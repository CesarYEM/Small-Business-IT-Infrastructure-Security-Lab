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

<img width="619" height="145" alt="image" src="https://github.com/user-attachments/assets/bbcf0c8e-3303-4c60-b0f9-d1bf36ee3e47" />

---

## 2. Automatización de Aprovisionamiento (OUs, Grupos y Usuarios)

Para garantizar la consistencia en el despliegue de identidades y reducir el margen de error humano asociado a la administración manual, se implementó un flujo automatizado mediante un script de PowerShell (`Create-ADStructure.ps1`) alimentado por un archivo plano (`usuarios.csv`).

### Estrategia de Control de Acceso (RBAC)
Se aplicó el modelo de Control de Acceso Basado en Roles (**RBAC**):
1. **Unidades Organizativas (OUs):** Estructuración jerárquica bajo la OU principal `CORP_Empresa` dividida por departamentos.
2. **Grupos de Seguridad:** Creación de grupos globales (`GRP_HR_RW`, `GRP_Ventas_RW`, `GRP_Admin_RW`) asignados a la OU `Grupos_Seguridad`.
3. **Membresía:** Los usuarios no reciben permisos directos a recursos; son asignados como miembros de los grupos departamentales correspondientes.

### Código del Script de Aprovisionamiento (`Create-ADStructure.ps1`)

```powershell
Import-Module ActiveDirectory

$DomainDN = "DC=corp,DC=cesartraining,DC=com"
$BaseOUName = "CORP_Empresa"
$BaseOUPath = "OU=$BaseOUName,$DomainDN"
$CSVPath = "C:\Users\Administrador\Documents\usuarios.csv"

$DefaultPassword = ConvertTo-SecureString "P4ssword2026!" -AsPlainText -Force

# 1. Creacion de OUs
if (-not (Get-ADOrganizationalUnit -Filter "Name -eq '$BaseOUName'")) {
    New-ADOrganizationalUnit -Name $BaseOUName -Path$DomainDN
}

$SubOUs = @("Recursos_Humanos", "Ventas", "Administracion", "Grupos_Seguridad", "Workstations", "Servidores")
foreach ($ou in $SubOUs) {$ouSearch = Get-ADOrganizationalUnit -Filter "Name -eq '$ou'" -SearchBase $BaseOUPath -ErrorAction SilentlyContinue
    if (-not $ouSearch) {
        New-ADOrganizationalUnit -Name $ou -Path$BaseOUPath
    }
}

# 2. Creacion de Grupos
$Groups = @("GRP_HR_RW", "GRP_Ventas_RW", "GRP_Admin_RW")
$GroupsOU = "OU=Grupos_Seguridad,$BaseOUPath"

foreach ($group in$Groups) {
    if (-not (Get-ADGroup -Filter "Name -eq '$group'")) {
        New-ADGroup -Name $group -GroupScope Global -GroupCategory Security -Path$GroupsOU
    }
}

# 3. Lectura de CSV y Creacion de Usuarios
$Users = Import-Csv -Path$CSVPath

foreach ($user in$Users) {
    $samAccount =$user.Username
    $userOU =$user.OU
    $groupName =$user.Group

    if (-not (Get-ADUser -Filter "SamAccountName -eq '$samAccount'")) {
        New-ADUser -Name "$($user.Firstname) $($user.Lastname)" -GivenName $user.Firstname -Surname$user.Lastname -SamAccountName $samAccount -UserPrincipalName "$samAccount@corp.cesartraining.com" -Path $userOU -AccountPassword$DefaultPassword -Enabled $true -ChangePasswordAtLogon$false
    }

    if ($groupName -and (Get-ADGroup -Filter "Name -eq '$groupName'")) {
        Add-ADGroupMember -Identity $groupName -Members$samAccount -ErrorAction SilentlyContinue
    }
}
```
---

## Integración del Cliente Linux (Ubuntu) en Active Directory

Se configuró el sistema cliente **Ubuntu Linux** para unirse al dominio de Active Directory (`corp.cesartraining.com`), logrando la resolución centralizada de identidades mediante la pila **SSSD**, **Kerberos** y **NSS**.

#### Arquitectura y Direccionamiento de Red
- **Controlador de Dominio (Windows Server):** `DC01.corp.cesartraining.com` (`192.168.100.222`)
- **Cliente Linux:** `cyem-ubuntu` (`192.168.100.x`)
- **Dominio / Realm:** `corp.cesartraining.com` / `CORP.CESARTRAINING.COM`
- **DNS Principal en Ubuntu:** `192.168.100.222`

---

## 1. Configuración Clave de Autenticación e Identidad

#### A. Protocolo Kerberos (`/etc/krb5.conf`)
Se definió la zona de autenticación indicando el Realm en mayúsculas y apuntando al Controlador de Dominio para la generación de tickets (`kinit`):

```ini
[libdefaults]
    default_realm = CORP.CESARTRAINING.COM
    dns_lookup_realm = false
    dns_lookup_kdc = true

[realms]
    CORP.CESARTRAINING.COM = {
        kdc = 192.168.100.222
        admin_server = 192.168.100.222
    }

[domain_realm]
    .corp.cesartraining.com = CORP.CESARTRAINING.COM
    corp.cesartraining.com = CORP.CESARTRAINING.COM
```

#### B. Servicio de Servicios de Seguridad del Sistema (/etc/sssd/sssd.conf)
Se configuró SSSD con mapeo de identidades POSIX automático (ldap_id_mapping = True) y deshabilitando el sufijo obligatorio (use_fully_qualified_names = False) para permitir consultas mediante nombres de usuario cortos:
```
[sssd]
domains = corp.cesartraining.com
config_file_version = 2
services = nss, pam

[domain/corp.cesartraining.com]
default_shell = /bin/bash
krb5_store_password_if_offline = True
cache_credentials = True
krb5_realm = CORP.CESARTRAINING.COM
realmd_tags = manages-system joined-with-adcli
id_provider = ad
access_provider = permit
fallback_homedir = /home/%u@%d
ad_domain = corp.cesartraining.com
use_fully_qualified_names = False
ldap_id_mapping = True
```
#### C. Name Service Switch (/etc/nsswitch.conf)
```
passwd:         files sss
group:          files sss
shadow:         files sss
```

## 2. Evidencia de Verificación y Resolución de Usuarios
Prueba de resolución del usuario de Active Directory cmendoza (Carlos Mendoza) asignado al grupo de seguridad GRP_HR_RW:

<img width="1215" height="169" alt="image" src="https://github.com/user-attachments/assets/649c1cc7-0c14-4a81-ac9a-a715b52f3d0e" />


## 3. Registro de Solución de Problemas (Troubleshooting Log)

# Error de Autenticación de Máquina (GSSAPI Error: Unspecified GSS failure):

Problema: El secreto guardado en /etc/krb5.keytab perdió desincronización con la cuenta de equipo (CYEM-UBUNTU$) tras múltiples intentos de unión.

Solución: Se desvinculó la máquina, se purgó la caché local (/var/lib/sss/db/*), y se re-generó el archivo de claves mediante sudo adcli join --domain=corp.cesartraining.com -U Administrador.

# Resolución por Nombre Corto:

Problema: El sistema solo reconocía al usuario pasando el FQDN completo (cmendoza@corp.cesartraining.com).

Solución: Se ajustó use_fully_qualified_names = False en sssd.conf y se ejecutó sudo sss_cache -E.
