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
- [DBeaver Community](https://dbeaver.io/download/) (Gestor de bases datos)
- [VI o Nano](https://docs.bluehosting.cl/tutoriales/servidores/guia-practica-de-los-editores-de-texto-nano-y-vi-en-linux.html) (Editor Texto Linux)

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
| MySQL | root | Laboratorio2025* | Puerto TCP: 3306 |
| PostgreSQL | postgres | Laboratorio2025* | Puerto TCP: 5432 |

<br>

---
# Paso a paso del laboratorio.

<br>

## Instalar VirtualBox
### Requisito previo para Windows: Instalar el Visual C++ Redistributable
- [Video Ejemplo #1](https://youtu.be/fvFNJiUs-ok)

### Instalación del VirtualBox
- [Video Ejemplo #2](https://youtu.be/yv35fmlpHqo)

### Crear Red NAT para el laboratorio
- [Video Ejemplo #3](https://youtu.be/1MMwK3n4xN8)

<br>

## Importar OVA en VirtualBox
- [Video Ejemplo #4](https://youtu.be/Qycnl7o-Arg)


<br>

## Instalar Zabbix 7.0 LTS
- [Video Ejemplo #5](https://youtu.be/bUZ83LE2kGE)

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

quit;
```

<br>

9. Activamos el servicio web del apache:
```bash
sudo systemctl enable apache2
sudo systemctl status apache2
```

<br>

10. Ajustamos configuración de PHP:
```bash
sudo nano /etc/php/8.3/cli/php.ini
```

<br>

11. Cambiamos los siguientes valores:
```bash
"Memory_Limit" a 2G
"max_execution_time" a 360
"date.timezone" a America/Bogota
"cgi.fix_pathinfo" a 0
```

<br>

12. Ahora si ajustamos el archivo WWW:
```bash
sudo nano /etc/php/8.3/fpm/pool.d/www.conf
```

<br>

13. Cambiamos el valor de "listen.allowed_clients" para que acepte conexiones de cualquier IP:
```bash
"listen.allowed_clients" a 0.0.0.0
```

<br>

14. Revisamos que el servicio de PHP esté activo:
```bash
sudo systemctl enable php8.3-fpm
sudo systemctl status php8.3-fpm
```

<br>

15. Descargamos e instalamos el ultimo repositorio LTS de ZABBIX - [Link](https://www.zabbix.com/download?zabbix=7.0&os_distribution=ubuntu&os_version=24.04&components=server_frontend_agent_2&db=mysql&ws=apache)
```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
apt update
```
⚠️ **Advertencia:** Para asegurar las ejecuciones debemos estar como "root".

<br>

16. Ahora instalamos el Zabbix Server, frontend y agent
```bash
apt install zabbix-server-mysql zabbix-frontend-php zabbix-apache-conf zabbix-sql-scripts zabbix-agent2
```

<br>

17. Ahora vamos a importar el esquema inicial y los datos. Debemos usar la clave de base de datos
```bash
zcat /usr/share/zabbix-sql-scripts/mysql/server.sql.gz | mysql --default-character-set=utf8mb4 -uzabbix -p zabbix
```

<br>

18. Nos conectamos nuevamente a MariaDB con nuestra contraseña
```bash
mysql -uroot -p
```

<br>

19. Desactivamos la opción log_bin_trust_function_creators tras importar el esquema de la base de datos:
```sql
set global log_bin_trust_function_creators = 0;
quit;
```

<br>

20. Editamos el archivo maestro de Zabbix para agregar la contraseña que creamos en los pasos ateriores
```bash
nano /etc/zabbix/zabbix_server.conf
```

<br>

21. Editamos el archivo maestro de Zabbix para agregar la contraseña que creamos en los pasos ateriores
```bash
DBPassword=Laboratorio2025*
```

<br>

22. Iniciar el servidor Zabbix y los procesos de agente
```bash
systemctl restart zabbix-server zabbix-agent2 apache2
systemctl enable zabbix-server zabbix-agent2 apache2
```

<br>

23. Revisamos la IP de nuestro servidor
```bash
ifconfig
```

<br>

24. Abrimos la pagina de Zabbix y seguimos el asistente para finalizar la instalación:
```bash
http://host/zabbix
```

<br>

25. Abrimos la pagina de Zabbix y seguimos el asistente para finalizar la instalación:
- Usuario: Admin
- Contraseña: zabbix

<br>

## Instalar Agente Zabbix en Ubuntu
- [Video Ejemplo #6](https://youtu.be/KOTntjxRVa0) - Instalar Agente
- [Video Ejemplo #7](https://youtu.be/F26Kvvfau8Y) - Crear Host

1. Nos conectamos como root: 
```bash
sudo -s
```

<br>

2. Instalamos el repositorio de Zabbix
```bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu24.04_all.deb
dpkg -i zabbix-release_latest_7.0+ubuntu24.04_all.deb
apt update
```

<br>

3. Instalamos el Zabbix agent 2
```bash
apt install zabbix-agent2 -y
```

<br>

4. Instalamos los plugins de, Zabbix agent 2
```bash
apt install zabbix-agent2-plugin-mongodb zabbix-agent2-plugin-mssql zabbix-agent2-plugin-postgresql
```

<br>

5. Ahora ajustamos el archivo de configuracion del Agent 2 de Zabbix para apuntarlo a nuestro servidor_
```bash
nano /etc/zabbix/zabbix_agent2.conf
```

<br>

6. Editamos el nombre del server y serveractive por la IP de nuestro servidor de Zabbix. En el Hostname asignamos el nombre de nuestro servidor
```bash
Server=192.168.10.100
ServerActive=192.168.10.100
Hostname=UBUNTU-LAB
```

<br>

7. Iniciamos el proceso del Zabbix agent 2
```bash
systemctl restart zabbix-agent2
systemctl enable zabbix-agent2
```

<br>

## Habilitar monitoreo MySQL
- [Video Ejemplo #8](https://youtu.be/wJt0gpoU6E4)

1. En el servidor que vamos a monitorear "UBUNTU-LAB" nos conectamos a MySQL con el usuario y contraseñas que se especificaron al principio de este documento:
```bash
 mysql -u root -p
```

<br>

2. Cree un usuario MySQL para la monitorización (Contraseña para el laboratorio: Laboratorio2025*):
```sql
CREATE USER 'zbx_monitor'@'%' IDENTIFIED BY 'Laboratorio2025*';
GRANT REPLICATION CLIENT, PROCESS, SHOW DATABASES, SHOW VIEW ON *.* TO 'zbx_monitor'@'%';
```

<br>

3. Indique en la macro {$MYSQL.DSN} el nombre de la fuente de datos de la instancia MySQL, ya sea el nombre de la sesión del archivo de configuración del agente 2 de Zabbix o la URI. 
- {$MYSQL.DSN}
```bash
tcp://192.168.10.102:3306
```

<br>

4. Defina el nombre de usuario y la contraseña en las macros de host ({$MYSQL.USER} y {$MYSQL.PASSWORD}).
- {$MYSQL.USER}
```bash
zbx_monitor
```
- {$MYSQL.PASSWORD}
```bash
Laboratorio2025*
```

<br>

## Habilitar el monitoreo en PostgreSQL
- [Video Ejemplo #9](https://youtu.be/X4bq7zg2wys)

1. Ingresamos a Postgres cambiando el usuario
```bash
sudo -i -u postgres
```

<br>

2. Nos conectamos a la base de datos
```bash
psql
```

<br>

3. Cree el usuario de PostgreSQL para la monitorización (<contraseña>, a su discreción) y herede los permisos del rol predeterminado pg_monitor:
```sql
CREATE USER zbx_monitor WITH PASSWORD 'Laboratorio2025*' INHERIT;
GRANT pg_monitor TO zbx_monitor;
\q
```

<br>

2. Agregamos la Plantilla al host y creamos las siguientes Macros:

- {$PG.URI}
```bash
tcp://192.168.10.102:5432
```

- {$PG.USER}
```bash
zbx_monitor
```

- {$PG.PASSWORD}
```bash
Laboratorio2025*
```

<br>

## Habilitar el monitoreo en Apache
- [Video Ejemplo #10](https://youtu.be/MCO1kepRJrg)

<br>

