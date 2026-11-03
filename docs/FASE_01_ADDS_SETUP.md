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

## 3. Automatización de Aprovisionamiento (OUs, Grupos y Usuarios)

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
