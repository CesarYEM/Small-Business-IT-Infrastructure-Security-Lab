# Fase 2: Control de Acceso Granular (PAM), Sudoers RBAC y Recursos Compartidos SMB

En esta segunda fase se implementaron controles de autorización a nivel de sistema operativo Linux y acceso a recursos compartidos en red desde el Controlador de Dominio (`DC01`), aplicando el principio de mínimo privilegio (RBAC) de extremo a extremo.

---

## 1. Creación Automática de Directorios Home (`pam_mkhomedir`)

Para evitar errores de sesión cuando un usuario de Active Directory inicia sesión por primera vez en el cliente Ubuntu (`CYEM-UBUNTU`), se habilitó el módulo PAM encargado de aprovisionar dinámicamente su directorio personal.

### Implementación:
```bash
sudo pam-auth-update
# Selección habilitada: [*] Create home directory on login
```
Asimismo, se ajustó la plantilla de ruta en /etc/sssd/sssd.conf (fallback_homedir = /home/%u) para generar rutas limpias (/home/<usuario>) sin el sufijo del dominio.

## 2. Filtrado de Inicio de Sesión por Grupos de Seguridad (SSSD / PAM)
Por defecto, cualquier cuenta autenticada en Active Directory puede iniciar sesión en un equipo unido al dominio. Para restringir el acceso únicamente al personal autorizado, se configuró el proveedor de acceso simple directamente en SSSD.

### Configuración aplicada (/etc/sssd/sssd.conf):

<img width="837" height="448" alt="image" src="https://github.com/user-attachments/assets/b25e4f0e-828b-4085-90e6-b50d3834adf1" />

### Validación de Seguridad (Pruebas Positiva y Negativa):

#### 1. Usuario Autorizado (cmendoza - miembro de GRP_HR_RW) y Usuario No Autorizado (Administrador / cuentas fuera de lista blanca):

<img width="446" height="187" alt="image" src="https://github.com/user-attachments/assets/84cb0b1e-e708-4bfa-bed1-60bea070ac69" />

## 3. Delegación de Privilegios Elevados (sudoers basado en RBAC)
Para administrar el sistema sin compartir contraseñas locales de root ni asignar permisos individuales por usuario, se delegó el uso de sudo exclusivamente al grupo de seguridad GRP_Admin_RW de Active Directory.
### Creación de regla modular (/etc/sudoers.d/ad-admins):

```bash
echo "%GRP_Admin_RW ALL=(ALL:ALL) ALL" | sudo tee /etc/sudoers.d/ad-admins
sudo chmod 440 /etc/sudoers.d/ad-admins
```
### Validación de Separación de Privilegios:
Usuario Estándar (cmendoza - GRP_HR_RW):

<img width="439" height="109" alt="image" src="https://github.com/user-attachments/assets/fe07d4b4-8f22-485b-af1e-ab2c1c48ebb6" />

Usuario Administrador Delegado (agomez - GRP_Admin_RW):

<img width="295" height="103" alt="image" src="https://github.com/user-attachments/assets/42dd3ad5-8440-47d4-b8f8-590e29f90cf3" />

## 4. Recurso Compartido por Red (SMB / CIFS) y Permisos NTFS

Se desplegó una carpeta departamental centralizada en Windows Server (C:\Recursos_Humanos) montada en Linux mediante el protocolo CIFS/SMB 3.0, respetando las identidades POSIX mapeadas desde Active Directory.

### A. Configuración en Windows Server (DC01):
1. Creación del directorio C:\Recursos_Humanos.
2. Permisos de Recurso Compartido (Share): Acceso delegado a GRP_HR_RW (Cambiar) y GRP_Admin_RW (Control total), eliminando el acceso predeterminado de Everyone.
3. Permisos NTFS (Seguridad): Asignación explícita de permisos de modificación/escritura para GRP_HR_RW y control total para GRP_Admin_RW.

### B. Integración y Montaje en Ubuntu (CYEM-UBUNTU):
``` bash
# Instalación de utilidades CIFS
sudo apt update && sudo apt install -y cifs-utils

# Creación del punto de montaje
sudo mkdir -p /mnt/recursos_humanos

# Montaje mapeando UID/GID de Active Directory
sudo mount -t cifs //192.168.100.222/Recursos_Humanos /mnt/recursos_humanos -o username=cmendoza,domain=corp.cesartraining.com,uid=$(id -u cmendoza),gid=$(id -g cmendoza),file_mode=0770,dir_mode=0770,vers=3.0
```

### C. Prueba de Lectura/Escritura Multiplataforma:

``` bash
sudo -u cmendoza touch /mnt/recursos_humanos/prueba_desde_ubuntu.txt
ls -l /mnt/recursos_humanos
# Resultado: Archivo creado exitosamente y visible en tiempo real en C:\Recursos_Humanos (DC01).
```
<img width="1129" height="588" alt="image" src="https://github.com/user-attachments/assets/d99069d4-d071-415a-b0da-4eda2dd8509d" />

