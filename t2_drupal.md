# Instalación de drupal en mi servidor local

### Configura el servidor web con virtual hosting para que el CMS sea accesible desde la dirección: www.nombrealumno-drupal.org

* Creamos el fichero de configuración para nuestro virtualhost y lo modificamos.

```sh
cd /etc/apache2/sites-avaliable
cp 000-default.conf celia-drupal.conf
nano celia-drupal.conf 
```
* Debe quedar de la siguiente forma:

```sh
<VirtualHost *:80>
        # The ServerName directive sets the request scheme, hostname and port that
        # the server uses to identify itself. This is used when creating
        # redirection URLs. In the context of virtual hosts, the ServerName
        # specifies what hostname must appear in the request's Host: header to
        # match this virtual host. For the default virtual host (this file) this
        # value is not decisive as it is used as a last resort host regardless.
        # However, you must set it for any further virtual host explicitly.

        ServerName www.celia-drupal.org
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/celia-drupal

        # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
        # error, crit, alert, emerg.
        # It is also possible to configure the loglevel for particular
        # modules, e.g.
        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        # For most configuration files from conf-available/, which are
        # enabled or disabled at a global level, it is possible to
        # include a line for only one particular virtual host. For example the
        # following line enables the CGI configuration for this host only
        # after it has been globally disabled with "a2disconf".
        #Include conf-available/serve-cgi-bin.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```

* Creamos el enlace simbólico

```sh
root@debian-cms:/etc/apache2/sites-available# a2ensite celia-drupal
Enabling site celia-drupal.
To activate the new configuration, you need to run:
  systemctl reload apache2
```

* Le damos los propietarios adecuados a nuestro documenroot (previamente creado).

```sh
chown -R www-data:www-data /var/www/celia-drupal
```




* Reiniciamos el servicio

```sh
systemctl reload apache2
```

### Crea un usuario en la base de datos para trabajar con la base de datos donde se van a guardar los datos del CMS. 

El usuario lo hemos creado antes en [este apartado](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/t1_lamp.md)

### Descarga la versión que te parezca más oportuna de Drupal y realiza la instalación.

* Tenemos que descargar Drupal desde la página oficial ya que, no hay paquetes en la distribución

```sh
wget https://www.drupal.org/download-latest/tar.gz -O drupal.tar.gz
```

* Para instalar drupal, descomprimimos el paquete en el documentroot donde vamos a tener nuestro sitio web

```sh
tar xf drupal.tar.gz -C /var/www/celia-drupal/
```

* Vemos el interior

```sh
root@debian-cms:/var/www/celia-drupal/drupal-9.0.7# ls
autoload.php   core		  INSTALL.txt  profiles    sites       vendor
composer.json  example.gitignore  LICENSE.txt  README.txt  themes      web.config
composer.lock  index.php	  modules      robots.txt  update.php

``` 
* Vamos a crear un enlace simbólico sobre el directorio de Drupal para tener un nombre sin números de versión.

```sh
ln -s /var/www/celia-drupal/drupal-9.0.7/ /var/www/celia-drupal/drupal
```

* Cambiamos el propietario para darle permisos al servidor web

```sh
chown -R www-data:www-data /var/www/celia-drupal/drupal
```

```sh
root@debian-cms:/var/www/celia-drupal# ls -l
total 4
lrwxrwxrwx 1 www-data www-data   35 Nov  1 17:08 drupal -> /var/www/celia-drupal/drupal-9.0.7/
drwxr-xr-x 8 www-data www-data 4096 Oct  7 19:48 drupal-9.0.7

```

* Drupal necesita algunos módulos de PHP extras que no hemos instalado anteriormente 

```sh
apt install php-apcu php-mbstring php-xml
```

* Reiniciamos el servidor de apache2

```sh
systemctl reload apache2
```

* Necesitamos habilitar url limpias, para ello editamos el fichero /etc/apache2/apache2.conf

```sh
#HABILITAR URL LIMPIAS

<Directory /var/www/celia-drupal>
         RewriteEngine On
         RewriteBase /
         RewriteCond %{REQUEST_FILENAME} !-f
         RewriteCond %{REQUEST_FILENAME} !-d
         RewriteRule ^(.*)$ index.php?q=$1 [L,QSA]
</Directory>

```

* Debemos activar el módulo **Rewrite** y algunos más que necesita Drupal de apache

```sh
root@debian-cms:~# a2enmod expires headers rewrite
Enabling module expires.
Enabling module headers.
Enabling module rewrite.
To activate the new configuration, you need to run:
  systemctl restart apache2

```
* Creamos una configuración para Drupal que permita el uso de archivos .htaccess que configure el módulo Rewrite

```sh
nano /etc/apache2/conf-available/drupal.conf
```

```sh
<Directory /var/www/html/drupal>
        AllowOverride all
</Directory>
```

* Activamos la configuración:

```sh
a2enconf drupal
```

* Por último reniciamos el servidor.

```sh
systemctl restart apache2
```

* Editamos el fichero /etc/hosts de nuestra máquina física

```sh
127.0.0.1       localhost
127.0.1.1       debian
192.168.100.162 www.departamentos.iesgn.org
192.168.100.160 www.celia-drupal.org
# The following lines are desirable for IPv6 capable hosts
::1     localhost ip6-localhost ip6-loopback
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters


```

* Ahora vamos a nuestro navegador y entramos a la siguiente dirección para instalar drupal:

www.celia-drupal.org/drupal

* Primero elegimos el idioma

![drupal1.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/drupal1.png)

* Pasamos a la selección de perfil de instalación, elegiremos estándar

![drupal2.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/drupal2.png)

* Vemos que cumplimos todos los requisitos

* Ahora nos muestra la configuración de la base de datos, debemos introducir el nombre de la base de datos que hemos creado anteriormente, el nombre del usuario que la usa y la contraseña para entrar a la base de datos. Guardamos y continuamos

![drupal3.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/drupal3.png)

* Después se empezará a instalar

![drupal4.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/drupal4.png)

* Cuando finaliza la instalación hay que configurar la identidad del sitio y crear un usuario que será el administrador

![drupal5.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/drupal5.png)


![drupal6.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/drupal6.png)

* Comprobamos que se ha instalado correctamente

![drupal7.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/drupal7.png)

### Realiza una configuración mínima de la aplicación (Cambia la plantilla, crea algún contenido, …)

