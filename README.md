# Small-Business-IT-Infrastructure-Security-Lab
Un laboratorio práctico que simula la infraestructura tecnológica y las operaciones de seguridad de una pequeña empresa. Diseñado para demostrar capacidades en administración de sistemas (Windows/Linux), servicios de red (Active Directory, DNS, SMB), políticas de grupo, respaldo de datos y monitoreo/detección de amenazas mediante SIEM.

## Índice de Documentación
- [Fase 1: Infraestructura Base & Directorio Activo](./docs/FASE_01_ADDS_SETUP.md)
- [Fase 2: Servicios de Red, Almacenamiento y GPO](./docs/FASE_02_SERVICES_GPO.md) *(Próximamente)*
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
