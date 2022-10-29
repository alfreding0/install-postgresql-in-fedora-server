28 Oct 2022

# Instalar PostgreSQL en Fedora Server 36
##### Notas
- La instalación será por línea de comandos desde la terminal. 
- Éste blog está realizado porque he tenido muchos problemas al instalarlo, por tanto, decidí registrarlo para ayudar a otras personas.
- La última versión stable que se instala a la fecha de hoy es PostgreSQL 14.


### Pasos a seguir
#### 1. Actualizar antes que nada.
Se actualizarán todos los paquetes y aplicaciones del sistema operativo. (Esperan a que termine de instalarse todo o puede que ser también que ya esté todo actualizado.)
`sudo dnf update -y && sudo dnf upgrade -y`

#### 2. Instalar postgresql (Solo el servidor de postgresql y su cliente de consola)
`sudo dnf install postgresql postgresql-server -y`

#### 3. Después de culminarse la instalación, intentamos conectarnos a postgresql.
`sudo -u postgres psql`

***Se mostrará un error como el siguiente:***
*could not change directory to "/root": Permission denied
psql: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: No such file or directory
Is the server running locally and accepting connections on that socket?*
**SI EN CASO NO TE SALIÓ ESTE ERROR, PUEDES PASAR DIRECTAMENTE AL [[PASO Nº7]][7].**

#### 4. Revisamos el estado del servicio de postgresql
`sudo systemctl status postgresql.service`

***Se mostrará lo siguiente:***
*\ postgresql.service - PostgreSQL database server
      Loaded: loaded (/usr/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
     Active: inactive (dead)*

#### 5. Intentamos iniciar el servicio de postgresql
`sudo systemctl start postgresql.service`

***Se mostrará lo siguiente:***
*Job for postgresql.service failed because the control process exited with error code.
See "systemctl status postgresql.service" and "journalctl -xeu postgresql.service" for details.
*

#### 6. Configuración según la guía oficial de [PostgreSQL](https://www.postgresql.org/download/linux/redhat/ "PostgreSQL")
`sudo postgresql-setup --initdb`

***Se mostrará lo siguiente:***
*Initializing database in '/var/lib/pgsql/data'
Initialized, logs are in /var/lib/pgsql/initdb_postgresql.log*

------------
`sudo systemctl enable postgresql.service`

***Se mostrará lo siguiente:***
*Created symlink /etc/systemd/system/multi-user.target.wants/postgresql.service → /usr/lib/systemd/system/postgresql.service.*

------------

`sudo systemctl start postgresql.service`

***No se mostrará ninguna salida***
*- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -*

#### 7. Ahora volvemos a ingresar al servidor de postgresql
`sudo -u postgres psql`

***Se mostrará lo siguiente:***
*could not change directory to "/root": Permission denied
psql (14.3)
Type "help" for help.
postgres=#*

#### 8. Hasta el paso anterior significa que el servidor de PostgreSQL está listo. Cambiemos la contraseña del super usuario de PostgreSQL "postgres"
Estando dentro del cliente de consola de postgresql (psql), escribir el comando \password, presionar enter e inmediatamente se mostrará la opción para ingresar la nueva contraseña, algo similar a lo siguiente:
postgres=# `\password`

*Enter new password for user "postgres":
Enter it again:
postgres=#*

------------

Salir de la consola de PostgreSQL (psql)
postgres=# `\quit`

#### 9. Conectarse a una base de datos utilizando el cliente "psql"
`psql -U postgres -d postgres -h localhost`

***Se mostrará lo siguiente:***
*psql: error: connection to server at "localhost" (::1), port 5432 failed: FATAL:  Ident authentication failed for user "postgres"*

#### 10. Tenemos que configurar los archivos de PostgreSQL. 
Para ello lo buscaremos en todo el sistema porque algunas veces podría cambiar la ubicación.
`sudo find / -iname "pg_hba.conf"`
***En este caso, se muestra lo siguiente:***
*/var/lib/pgsql/data/pg_hba.conf*
En caso de tener otro resultado, manejar ese resultado para ustedes.

#### 11. Realizamos una copia de la configuración original.
Tener en cuenta que la ruta del archivo, es según el resultado del paso anterior.
`sudo cp /var/lib/pgsql/data/pg_hba.conf /var/lib/pgsql/data/pg_hba.conf.copy`

#### 12. Editar las formas de acceso para la conexión al servidor.
`sudo vim /var/lib/pgsql/data/pg_hba.conf`

Aproximadamente entre la línea 80 y 90, que están casi al final, encontrar algo similar a lo siguiente:

Antes
![Antes](/screenshots/pg_hba.conf-antes.jpg)

-----------------------

Después
![Después](/screenshots/pg_hba.conf-despues.jpg)

#### 13. Recargar las configuraciones del servicio
`sudo systemctl reload postgresql.service`

#### 14. Revisar estado del servicio
`sudo systemctl status postgresql.service`

***Se mostrará lo siguiente:***
*postgresql.service - PostgreSQL database server
     Loaded: loaded (/usr/lib/systemd/system/postgresql.service; enabled; vendor preset: disabled)
     Active: active (running) since Sat 2022-10-29 03:11:27 UTC; 27min ago
    Process: 14277 ExecStartPre=/usr/libexec/postgresql-check-db-dir postgresql (code=exited, status=0/SUCCESS)
    Process: 14493 ExecReload=/bin/kill -HUP $MAINPID (code=exited, status=0/SUCCESS)
   Main PID: 14279 (postmaster)
      Tasks: 8 (limit: 9492)
     Memory: 18.8M
        CPU: 264ms
     CGroup: /system.slice/postgresql.service
             ├─14279 /usr/bin/postmaster -D /var/lib/pgsql/data
             ├─14280 "postgres: logger "
             ├─14282 "postgres: checkpointer "*

#### 15. Volvemos a conectarse mediante "psql"
`psql -U postgres -d postgres -h localhost`

***Se mostrará lo siguiente:***
*Password for user postgres:
psql (14.3)
Type "help" for help.
postgres=#*

Listo!!! Hemos logrado instalar y configurar el servicio de PostgreSQL.

**Recomendado!**
- Comandos para utilizar la terminal **psql**, el cliente consola de PostgreSQL. (Pronto)
- Instalar PgAdmin4, el cliente gráfico de PostgreSQL. (Pronto)
