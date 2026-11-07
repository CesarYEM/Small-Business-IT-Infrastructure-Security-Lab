# Small-Business-IT-Infrastructure-Security-Lab

> **Project Status:** 🟢 **Phase 1 Complete** (Active Directory + Ubuntu integration) | 🟡 **Phase 2 In Progress** (PAM access control & SMB shares)

---

## 🇬🇧 Summary

Hands-on homelab that simulates the IT infrastructure and security operations of a small business. It demonstrates the integration of an Ubuntu Linux client into a Windows Server Active Directory domain (`corp.cesartraining.com`), with centralized identity management, RBAC enforced through PowerShell automation, Kerberos authentication and SSSD/NSS identity mapping. The long-term goal is to extend the lab with backups, hardening and SIEM-based detection.

**Key skills demonstrated so far**

- **Active Directory & identity management:** OU design, security groups and automated user provisioning with PowerShell (`Create-ADStructure.ps1`).
- **Linux authentication:** realm join, SSSD configuration, Kerberos ticket management and NSS identity mapping.
- **Troubleshooting:** diagnosing a Kerberos GSSAPI failure caused by an out-of-sync keytab, rebuilding it, and fixing short-name user resolution in SSSD (documented step by step in the Phase 1 doc).

> The lab runs in an isolated environment. The domain, users and credentials are fictional.

---

## 🇪🇸 Descripción

Laboratorio práctico que simula la infraestructura de TI y las operaciones de seguridad de una pequeña empresa. Está pensado para demostrar capacidades en administración de sistemas (Windows/Linux), servicios de red (Active Directory, DNS, SMB), políticas de grupo, respaldo de datos y, a futuro, monitoreo y detección de amenazas con un SIEM.

### Escenario

Se simula el entorno de TI de una pequeña empresa en crecimiento con tres departamentos:

- **Recursos Humanos**
- **Ventas**
- **Administración**

### Objetivos

- Centralizar identidades, autenticación y políticas con Active Directory.
- Aplicar control de acceso basado en el principio de mínimo privilegio (RBAC/NTFS).
- Desplegar servicios esenciales en Linux y Windows Server.
- Configurar recolección de eventos y monitoreo centralizado de seguridad.

---

## 📐 Arquitectura de red

Red del laboratorio: `192.168.100.0/24`

| Host / VM | Sistema operativo | Rol / servicios | IP | Estado |
| :--- | :--- | :--- | :--- | :--- |
| **DC01** | Windows Server 2025 | Active Directory, DNS | 192.168.100.222 (estática) | ✅ Operativo |
| **cyem-ubuntu** | Ubuntu Linux | Cliente unido al dominio (SSSD / Kerberos) | 192.168.100.x | ✅ Operativo |
| **WS01** | Windows 10/11 Pro | Estación de trabajo (RH / Ventas) | DHCP | 🕒 Planeado |
| **SRV-LNX01** | Ubuntu Server 22.04 | Nginx / agente SIEM / recolector de logs | 192.168.100.50 | 🕒 Planeado |

---

## 🗺️ Roadmap

- [x] **Fase 1:** Active Directory, estructura de OUs y grupos, aprovisionamiento con PowerShell (RBAC) e integración del cliente Ubuntu con SSSD y Kerberos
- [ ] **Fase 2 (en progreso):** Control de acceso con PAM, recursos compartidos SMB y GPO
- [ ] **Fase 3:** Respaldos, hardening y monitoreo con SIEM
- [ ] **Fase 4:** Detección de incidentes y análisis de logs

---

## 📚 Documentación

| Fase | Documento |
| :--- | :--- |
| Fase 1: Infraestructura base e integración Linux | [docs/FASE_01_ADDS_SETUP.md](./docs/FASE_01_ADDS_SETUP.md) |
| Fase 2 | En preparación |
| Fase 3 | En preparación |
| Fase 4 | En preparación |

---

## 🛠️ Tecnologías

Windows Server 2025 · Active Directory Domain Services · DNS · PowerShell · Ubuntu · SSSD · Kerberos · NSS · realmd/adcli
