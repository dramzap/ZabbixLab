# Laboratorio de Zabbix - FLISOL COLOMBIA 2025
El presente laboratorio tiene como objetivo dar una guia rápida para instalar Zabbix en su versión 7.0 en un servidor Ubuntu 24.04 y habilitar el monitoreo en un segundo servidor que tiene expuesto el servicio Web de Apache y de bases de datos MySQL y PostgreSQL.

Para lograr ejecutar el laboratorio se recomienda contar como mínimo con los siguientes requisitos mínimos
| Recurso | Cantidad/Tipo |
|----------|----------|
| CPU | 4 VCORE |
| RAM | 8 GB |
| Espacio libre de disco | 20 GB |
| Sistema Operativo | Windows, Linux o MacOS |

<br>

Adicionalmente debemos contar con:
- Conexión a Internet
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads) (Software de virtualización)
- [Descargar OVA](https://drive.google.com/drive/folders/17459DXlpWFOcxB-hEpSLbz0lO1ZAKFCV?usp=drive_link) (Este contiene las VM que vamos a importar a VirtualBox)

<br>

Opcionalmente también pueden descargar las siguientes herramientas complementarias:
- [Putty](https://www.putty.org) (Cliente SSH)
- [mRemoteNG](https://mremoteng.org/download) (Cliente RDP y SSH)

<br>

Para nuestro laboratorio vamos a contar con las siguientes máquinas virtuales:
| Nombre del Servidor | IP |
|----------|----------|
| ZABBIX-LAB | 192.168.10.100 |
| UBUNTU-LAB | 192.168.10.102 |

<br>

Las siguientes son las cuentas de usuario para usar en nuestro laboratorio:
| Tipo | Usuario | Contraseña | Observaciones |
|----------|----------|----------|----------|
| Sistema Operativo | dba | Laboratorio2025* | N/A |
| MySQL | dba | Laboratorio2025* | Puerto TCP: 3306 |
| PostgreSQL | postgres | Laboratorio2025* | Puerto TCP: 5432 |




---






