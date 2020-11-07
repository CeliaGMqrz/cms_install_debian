# Configuración multinodo


### 1. Realiza un copia de seguridad de la base de datos

Vamos a utilizar el comando **mysqldump**, que se trata de un programa ejecutable desde la línea de comandos que permite realizar una **exportación completa**(dump), de todo el contenido de una base de datos. ESte comando tiene multitud de parámetros distintos. 

Para realizar una exportación completa de toda la información de la base de datos que incluya estructura, datos, eventos, procedimientos, triggers y vistas, utilizaremos lo siguiente:


```sh
$ mysqldump -v --opt --events --routines --triggers --default-character-set=utf8 -u your_username -p your_db_name > db_backup_your_db_name_`date +%Y%m%d_%H%M%S`.sql | gzip -c
```

Con **| gzip -c**, lo que hacemos es guardar la copia de seguridad en un fichero comprimido.

Es recomendable crear un usuario solo para hacer copias de seguridad, nos metemos como root y le damos los priviegios a este usuario nuevo

```sh
mysql -u root -p
```

```sh
mysql> GRANT LOCK TABLES, SELECT ON *.* TO 'cgm_backup'@'%' IDENTIFIED BY '*****';
```
```sh
grant all privileges on cms.* to 'cgm_backup'@'localhost' identified by '*****';
quit
```

* Ahora ejecutamos el comando para crear la copia de seguridad:

```sh
vagrant@debian-cms:~$ mysqldump -v --opt --events --routines --triggers --default-character-set=utf8 -u cgm_backup -p cms > db_backup_cms_`date +%Y%m%d_%H%M%S`.sql

```
* Vemos que se ha generado 

```sh
vagrant@debian-cms:~$ ls
db_backup_cms_20201107_121456.sql

```

### 2. Crea otra máquina con vagrant, conectada con una red interna a la anterior y configura un servidor de base de datos.


```sh
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
        config.vm.define "debian_cms" do |servidor|
                config.vm.box = "debian/buster64"
                config.vm.hostname = "debian-cms"
                config.vm.network :public_network, :bridge=>"enp5s0"
                config.vm.network "private_network",
                        virtualbox__intnet: "local_cms", ip: "10.0.0.2"
        end
        config.vm.define "cliente" do |cliente|
                cliente.vm.box = "debian/buster64"
                cliente.vm.hostname = "cliente"
                cliente.vm.network "private_network",
                        virtualbox__intnet: "local_cms", ip: "10.0.0.3"
        end
end
```
Comprobamos que tienen conexión el servidor con el cliente

**Servidor-cliente**

```sh
root@debian-cms:/home/vagrant# ping 10.0.0.3
PING 10.0.0.3 (10.0.0.3) 56(84) bytes of data.
64 bytes from 10.0.0.3: icmp_seq=1 ttl=64 time=0.833 ms
64 bytes from 10.0.0.3: icmp_seq=2 ttl=64 time=0.881 ms
64 bytes from 10.0.0.3: icmp_seq=3 ttl=64 time=0.802 ms
^C
--- 10.0.0.3 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 5ms
rtt min/avg/max/mdev = 0.802/0.838/0.881/0.046 ms


```

**Cliente-servidor**

```sh
root@cliente:/home/vagrant# ping 10.0.0.2
PING 10.0.0.2 (10.0.0.2) 56(84) bytes of data.
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.233 ms
64 bytes from 10.0.0.2: icmp_seq=2 ttl=64 time=0.588 ms
^C
--- 10.0.0.2 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 63ms
rtt min/avg/max/mdev = 0.233/0.410/0.588/0.178 ms

```

**Configurar servidor de base de datos**

* Instalamos mysql

```sh
apt install mariadb-client mariadb-server
```

* mysql en funcionamiento

```sh
root@cliente:/home/vagrant# systemctl status mariadb
● mariadb.service - MariaDB 10.3.25 database server
   Loaded: loaded (/lib/systemd/system/mariadb.service; enabled; vendor preset: enabled)
   Active: active (running) since Sat 2020-11-07 12:58:21 GMT; 1min 10s ago
     Docs: man:mysqld(8)
           https://mariadb.com/kb/en/library/systemd/
 Main PID: 2330 (mysqld)
   Status: "Taking your SQL requests now..."
    Tasks: 31 (limit: 544)
   Memory: 70.1M
   CGroup: /system.slice/mariadb.service
           └─2330 /usr/sbin/mysqld

Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: Running 'mysqlcheck' with connection arguments: --
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: # Connecting to localhost...
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: # Disconnecting from localhost...
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: Processing databases
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: information_schema
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: performance_schema
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: Phase 7/7: Running 'FLUSH PRIVILEGES'
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2368]: OK
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2463]: Checking for insecure root accounts.
Nov 07 12:58:21 cliente /etc/mysql/debian-start[2470]: Triggering myisam-recover for all MyISAM tables an
lines 1-22/22 (END)

```

### 3. Crea un usuario en la base de datos para trabajar con la nueva base de datos.

* Creamos el usuario nuevo

```sh

root@cliente:/home/vagrant# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 49
Server version: 10.3.25-MariaDB-0+deb10u1 Debian 10

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create user 'cliente1'@'localhost';
Query OK, 0 rows affected (0.001 sec)


```

* Le damos permisos sobre la nueva base de datos

```sh
MariaDB [(none)]> grant all privileges on cms_cliente.* to 'cliente1'@'localhost' identified by '********';
```



### 4. Restaura la copia de seguridad en el nuevo servidor de base datos.

Para restaurar la base de datos tenemos que enviar el fichero a nuestro cliente desde el servidor.

Previamente se ha añadido la clave pública del servidor en el fichero authorized keys del cliente para poder pasar la copia de seguridad.

* Se lo enviamos al cliente desde el servidor:

```sh
vagrant@debian-cms:~$ scp db_backup_cms_20201107_121456.sql vagrant@10.0.0.3:/home/vagrant
db_backup_cms_20201107_121456.sql                                      100%   11MB  90.3MB/s   00:00 
```

* Comprobamos que la tenemos en el directorio personal del cliente

```sh
vagrant@cliente:~$ ls
db_backup_cms_20201107_121456.sql

```
* Para restaurarla hacemos lo siguiente:

```sh
mysql -u usuario --pasword=contraseña_usuario nombrebd < backup.sql
```

* Quedaría así:

```sh
vagrant@cliente:~$ mysql -u cliente1 --password=cliente1 cms_cliente < db_backup_cms_20201107_121456.sql
```

* Entramos como cliente1 y comprobamos que se ha restaurado la copia de seguridad correctamente

```sh
vagrant@cliente:~$ mysql -u cliente1 -p
```

```sh
MariaDB [(none)]> use cms_cliente
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
MariaDB [cms_cliente]> show tables;
+-------------------------------------------------+
| Tables_in_cms_cliente                           |
+-------------------------------------------------+
| batch                                           |
| block_content                                   |
| block_content__body                             |
| block_content__field_clean_blog_header_backgrou |
| block_content__field_clean_blog_header_image    |
| block_content_field_data                        |
| block_content_field_revision                    |
| block_content_r__846f9b8ab9                     |
| block_content_r__93aa615698                     |
| block_content_revision                          |
| block_content_revision__body                    |
| cache_bootstrap                                 |
| cache_config                                    |
| cache_container                                 |
| cache_data                                      |
| cache_default                                   |
| cache_discovery                                 |
| cache_dynamic_page_cache                        |
| cache_entity                                    |
| cache_menu                                      |
| cache_page                                      |
| cache_render                                    |
| cache_toolbar                                   |
| cachetags                                       |
| comment                                         |
| comment__comment_body                           |
| comment_entity_statistics                       |
| comment_field_data                              |
| config                                          |
| file_managed                                    |
| file_usage                                      |
| flood                                           |
| history                                         |
| key_value                                       |
| key_value_expire                                |
| locale_file                                     |
| locales_location                                |
| locales_source                                  |
| locales_target                                  |
| menu_link_content                               |
| menu_link_content_data                          |
| menu_link_content_field_revision                |
| menu_link_content_revision                      |
| menu_tree                                       |
| node                                            |
| node__body                                      |
| node__comment                                   |
| node__field_image                               |
| node__field_tags                                |
| node_access                                     |
| node_field_data                                 |
| node_field_revision                             |
| node_revision                                   |
| node_revision__body                             |
| node_revision__comment                          |
| node_revision__field_image                      |
| node_revision__field_tags                       |
| path_alias                                      |
| path_alias_revision                             |
| queue                                           |
| router                                          |
| search_dataset                                  |
| search_index                                    |
| search_total                                    |
| semaphore                                       |
| sequences                                       |
| sessions                                        |
| shortcut                                        |
| shortcut_field_data                             |
| shortcut_set_users                              |
| taxonomy_index                                  |
| taxonomy_term__parent                           |
| taxonomy_term_data                              |
| taxonomy_term_field_data                        |
| taxonomy_term_field_revision                    |
| taxonomy_term_revision                          |
| taxonomy_term_revision__parent                  |
| user__roles                                     |
| user__user_picture                              |
| users                                           |
| users_data                                      |
| users_field_data                                |
| votingapi_result                                |
| votingapi_vote                                  |
| watchdog                                        |
+-------------------------------------------------+
85 rows in set (0.001 sec)

```
### 5. Desinstala el servidor de base de datos en el servidor principal.

* Paramos el servicio

```sh
/etc/init.d/mysql stop
/etc/init.d/mariadb stop

```

* Eliminamos todo en cuanto a mysql y mariadb

```sh
apt-get remove --purge mariadb*
apt-get remove --purge mysql*
apt-get autoremove
```


* Comprobamos que se ha eliminado

```sh
root@debian-cms:/home/vagrant# systemctl status mariadb
Unit mariadb.service could not be found.
root@debian-cms:/home/vagrant# systemctl status mysql
Unit mysql.service could not be found.

```

### 6. Realiza los cambios de configuración necesario en drupal para que la página funcione.


Como hemos eliminado el gestor de base datos en el servidor ahora la web no funciona. Tenemos que modificar un fichero de configuración que es **settings.php** que se encuentra en nuestro virtualhosting del servidor. 

Nos vamos al final del fichero y vemos la configuración de las bases de datos que vamos a modificar, actualmente tiene este aspecto que apuntaba a nuestra anterior base de datos que hemos eliminado

```sh
vagrant@debian-cms:/var/www/celia-drupal/drupal-9.0.7/sites/default$ sudo nano settings.php 
```

```sh
$databases['default']['default'] = array (
  'database' => 'cms',
  'username' => 'cgm',
  'password' => 'cgm',
  'prefix' => '',
  'host' => 'localhost',
  'port' => '3306',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' => 'mysql',
);
```

La modificamos para que apunte a nuestro cliente donde está restaurada:

```sh
$databases['default']['default'] = array (
  'database' => 'cms_cliente',
  'username' => 'cliente1',
  'password' => 'cliente1',
  'prefix' => '',
  'host' => '10.0.0.3',
  'port' => '3306',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' => 'mysql',
);
```

Guardamos la nueva configuración y reiniciamos el servicio de apache

```sh
systemctl restart apache2
```

