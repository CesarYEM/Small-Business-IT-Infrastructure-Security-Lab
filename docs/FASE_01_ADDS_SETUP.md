[← Volver al README Principal](../README.md)

# Fase 1: Infraestructura Base & Instalación de AD DS

## 1. Configuración de Red y Justificación Técnica

Para el correcto funcionamiento de los servicios de directorio en un entorno corporativo, la interfaz de red del Controlador de Dominio (`DC01`) requiere una configuración IP estática.

### Parámetros de Red Asignados
* **Interfaz de Red (Alias):** `External`
* **Perfil de Red (Dominio):** `corp.cesartraining.com`
* **Dirección IPv4:** `192.168.100.222`
* **Máscara de Subred:** `/24` (`255.255.255.0`)
* **Puerta de Enlace Predeterminada (Gateway):** `192.168.100.1`
* **Servidor DNS Primario:** `127.0.0.1` (Loopback local)

### Justificación Técnica de Arquitectura
1. **Asignación de IP Estática:** Un Controlador de Dominio (DC) provee servicios de autenticación (Kerberos/NTLM) y resolución de nombres (DNS) críticos a toda la infraestructura. Utilizar DHCP en un DC introduciría el riesgo de que la dirección IP cambie tras un reinicio, interrumpiendo los inicios de sesión de los clientes y el acceso a recursos compartidos SMB.
2. **DNS Apuntando a Loopback (`127.0.0.1`):** Active Directory depende intrínsecamente del servicio DNS para la localización de servicios (registros SRV). Configurar el DNS preferido en `127.0.0.1` (o la propia IP `192.168.100.222`) garantiza que el servidor consulte primero su propia base de datos integrada de AD-DNS antes de intentar resolver peticiones externas.

### Evidencia de Configuración
Salida del comando `Get-NetIPConfiguration` en PowerShell:

![Configuración de Red IP y DNS](<img width="619" height="145" alt="image" src="https://github.com/user-attachments/assets/bbcf0c8e-3303-4c60-b0f9-d1bf36ee3e47" />
)
