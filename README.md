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
- [Visual C++ Redistributable](https://learn.microsoft.com/es-es/cpp/windows/latest-supported-vc-redist?view=msvc-170) (Sólo para Windows)

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

<br>

---
# Paso a paso del laboratorio.

<br>

## Instalar VirtualBox

<br>

## Importar OVA en VirtualBox

<br>

## Instalar Zabbix 7.0 LTS
1. Ingresamos a nuestro servidor "ZABBIX-LAB" y nos habilitamos como root para tener permisos de instalación (Nos va a requerir la contraseña):
```bash
sudo su root
```

<br>

2. Ahora debemos asegurar que distribución y versión de Linux estamos usando:
```bash
cat /etc/os-release
```

<br>

3. Con el siguiente comando actualizamos la lista de paquetes y se instalan las últimas versiones disponibles en el sistema sin pedir confirmación:
```bash
sudo apt update -y
sudo apt upgrade -y
```

<br>

4. Instalamos el servicio de Chrony para mantener la hora actualizada del servidor:
```bash
sudo apt install chrony -y
sudo systemctl enable chrony
sudo systemctl status chrony
sudo timedatectl set-timezone America/Bogota
timedatectl status
```

<br>

5. Instalamos todos los prerequisitos:
```bash
sudo apt install mariadb-server php php-cli php-common php-fpm php-curl php-mysql apache2 curl  -y

sudo systemctl enable mariadb
sudo systemctl status mariadb
```

<br>

6. Con el sigueinte comando vamos a mejorar la seguridad de la instancia de base de datos configurando una contraseña para root, eliminando accesos inseguros y aplicando cambios recomendados:
```bash
sudo mysql_secure_installation
```
⚠️ **Advertencia:** Solo para fines educativos, sugiero que asignemos la misma contraseña que estamos manejando para todo el laboratorio. En entornos corporativos asignar contraseñas seguras.

<br>

7. Ahora nos conectamos a la base de datos con la contraseña que escogimos:
```bash
sudo mysql -u root -p
```
<br>

8. Ya que estamos conectado en MariaDB, vamos a crear la base de datos y usuario para Zabbix:
```sql
create database zabbix character set utf8mb4 collate utf8mb4_bin;
create user zabbix@localhost identified by 'Laboratorio2025*';
grant all privileges on zabbix.* to zabbix@localhost;

set global log_bin_trust_function_creators = 1;

flush privileges;

exit
```

<br>

9. Activamos el servicio web del apache:
```bash
sudo systemctl enable apache2
sudo systemctl status apache2
```

<br>
