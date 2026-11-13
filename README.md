# Active Directory & Linux Integration Lab (SSSD / Kerberos / RBAC)

> **Project Status:** 🟢 **Phase 1 Complete** (AD & Ubuntu Integration) | 🟡 **Phase 2 In Progress** (PAM Access Control & Shares)

---

### 🇬🇧 Project Overview
Enterprise-oriented homelab project demonstrating the integration of an Ubuntu Linux client into a Windows Server Active Directory domain (`corp.cesartraining.com`). Focused on centralized identity management, RBAC enforcement using PowerShell automation, Kerberos authentication, and SSSD/NSS identity mapping. Designed to reflect real-world SOC and System Administration scenarios.

### 🇪🇸 Habilidades Demostradas (Key Skills)
- **Active Directory & Identity Management:** Design of OUs, Security Groups, and automated user provisioning via PowerShell (`Create-ADStructure.ps1`).
- **Linux Security & Authentication:** Realm joining, SSSD configuration, Kerberos ticket management, and NSS database mapping.
- **Troubleshooting & Auditing:** Resolution of Kerberos GSSAPI desynchronization, keytab regeneration, and DNS/NTP alignment.

---

# Small-Business-IT-Infrastructure-Security-Lab
Un laboratorio práctico que simula la infraestructura tecnológica y las operaciones de seguridad de una pequeña empresa. Diseñado para demostrar capacidades en administración de sistemas (Windows/Linux), servicios de red (Active Directory, DNS, SMB), políticas de grupo, respaldo de datos y monitoreo/detección de amenazas mediante SIEM.

## Índice de Documentación
- [Fase 1: Infraestructura Base & Directorio Activo](./docs/FASE_01_ADDS_SETUP.md)
- [Fase 2: Servicios de Red, Almacenamiento y GPO](./docs/FASE_02_SERVICES_GPO.md)
- [Fase 3: Respaldos, Hardening y Monitoreo SIEM](./docs/FASE_03_BACKUP_SIEM.md) *(Próximamente)*
- [Fase 4: Detección de Incidentes y Análisis de Logs](./docs/FASE_04_SOC_DETECTION.md) *(Próximamente)*

## Escenario
Se simula el entorno de TI de una pequeña empresa en crecimiento con tres departamentos activos:
- **Recursos Humanos**
- **Ventas**
- **Administración**

### Objetivos del Proyecto:
- Centralizar la gestión de identidades, autenticación y políticas con Active Directory.
- Implementar control de acceso basado en el principio de mínimo privilegio (RBAC/NTFS).
- Desplegar servicios esenciales en Linux y Windows Server.
- Configurar recolección de eventos y monitoreo centralizado de seguridad.

## 📐 Arquitectura de Red y Subnetting

| Host / VM | Sistema Operativo | Rol / Servicios | IP / Subnet |
| :--- | :--- | :--- | :--- |
| **DC01** | Windows Server 2025 | Active Directory, DNS Server | 192.168.100.222/24 |
| **WS01** | Windows 10/11 Pro | Cliente Workstation (HR/Ventas) | DHCP (192.168.100.x) |
| **SRV-LNX01** | Ubuntu Server 22.04 | Nginx / Agente SIEM / Log Collector | 192.168.100.50/24 |

---

## Medida de Hardening y Zero Trust: Restricción de Unión de Equipos (Join Control)

Por defecto en Active Directory, el atributo `ms-DS-MachineAccountQuota` permite que cualquier usuario autenticado sin privilegios una hasta 10 computadoras al dominio. Para alinearse a los principios de **Zero Trust (Confianza Cero)** y **Menor Privilegio**, se eliminó esta capacidad predeterminada.

### 1. Modificación de la Cuota en Active Directory (GUI)
1. En el Controlador de Dominio, se abrió la consola **Usuarios y equipos de Active Directory** (`dsa.msc`) habilitando las **Características avanzadas**.
2. En las **Propiedades** de la raíz del dominio (`corp.cesartraining.com`), desde el **Editor de atributos**, se localizó la propiedad `ms-DS-MachineAccountQuota`.
3. Se modificó el valor predeterminado de `10` a `0`.

### 2. Prueba de Denegación de Acceso (Validación de Seguridad)
Se ejecutó un intento de enrolamiento desde el cliente Linux utilizando la cuenta de un usuario estándar sin privilegios administrativos (`cmendoza`):

```bash
sudo realm join --verbose -U cmendoza corp.cesartraining.com
```
<img width="1213" height="702" alt="image" src="https://github.com/user-attachments/assets/a3ff3c2a-b4d5-45fa-8297-5424c1e6e03a" />

