# Tarea 4: Instalación de otro CMS 

### Elige otro CMS realizado en PHP y realiza la instalación en tu infraestructura.

Vamos a instalar **'Moodle'**.

#### Instalación de Moodle

Requisitos:

* Tener instalado un entorno [LAMP](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/t1_lamp.md)

NOTA: necesitaremos otros módulos de php para instalar moodle

```sh
apt-get install php.xml php.zip php.curl php-gd php.intl php.soap php.mbstring

```

Antes de instalar Moodle necesitamos crear un virtualhost en apache2.

Creamos el fichero de configuración 

```sh
root@server:/etc/apache2/sites-available# nano moodlecg.conf 
```

Lo dejamos así:

```sh
<VirtualHost *:80>

        ServerName www.moodlecg.iesgn.org
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/moodle
        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>
```

Lo activamos:

```sh
root@server:/etc/apache2/sites-available# a2ensite moodlecg
Enabling site moodlecg.
To activate the new configuration, you need to run:
  systemctl reload apache2
```

Creamos el directorio 'moodle' en /var/www/ y le damos el propietario adecuado.

```sh
root@server:/var/www# mkdir moodle
root@server:/var/www# chown -R www-data:www-data moodle
root@server:/var/www# ls -l
total 8
drwxr-xr-x 2 root     root     4096 Nov  8 14:29 html
drwxr-xr-x 2 www-data www-data 4096 Nov  8 14:57 moodle

```

Descargamos Moodle y lo enviamos a nuestro servidor

```sh
celiagm@debian:~/Descargas$ scp moodle-3.9.2.zip vagrant@192.168.100.177:/home/vagrant
The authenticity of host '192.168.100.177 (192.168.100.177)' can't be established.
ECDSA key fingerprint is SHA256:DuOJ1cJ0Q6/7IDnVYI5Wh0SNwpsQP+RQAEEHVNybN3U.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.100.177' (ECDSA) to the list of known hosts.
moodle-3.9.2.zip                                                       100%   71MB 162.0MB/s   00:00  
```
Descomprimimos el paquete

```sh
unzip moodle-3.9.2.zip
```
Vemos que se ha descompimido correctamente

```sh
root@server:/home/vagrant# ls
moodle	moodle-3.9.2.zip

```
Movemos la capeta moodle a nuestro documentroot

```sh
root@server:/home/vagrant# mv moodle /var/www/
root@server:/home/vagrant# cd /var/www/moodle/
root@server:/var/www/moodle# ls
admin				      enrol		      my
analytics			      error		      notes
auth				      favourites	      npm-shrinkwrap.json
availability			      file.php		      package.json
babel-plugin-add-module-to-define.js  files		      phpunit.xml.dist
backup				      filter		      pix
badges				      githash.php	      plagiarism
behat.yml.dist			      grade		      pluginfile.php
blocks				      group		      portfolio
blog				      GruntfileComponents.js  privacy
brokenfile.php			      Gruntfile.js	      PULL_REQUEST_TEMPLATE.txt
cache				      h5p		      question
calendar			      help_ajax.php	      rating
cohort				      help.php		      README.txt
comment				      index.php		      report
competency			      install		      repository
completion			      install.php	      rss
composer.json			      INSTALL.txt	      search
composer.lock			      iplookup		      tag
config-dist.php			      lang		      theme
contentbank			      lib		      tokenpluginfile.php
CONTRIBUTING.txt		      local		      TRADEMARK.txt
COPYING.txt			      login		      user
course				      media		      userpix
customfield			      message		      version.php
dataformat			      mnet		      webservice
draftfile.php			      mod

```

Nos aseguramos de que tenga el propietario adecuado

```sh
root@server:/var# chown -R www-data:www-data www/
root@server:/var/www# chown -R www-data:www-data moodle/
root@server:/var/www# ls -l
total 8
drwxr-xr-x  2 root     root     4096 Nov  8 14:29 html
drwxr-xr-x 56 www-data www-data 4096 Sep 11 04:12 moodle

```
Configuramos el /etc/host de nuestra máquina anfitriona

```sh
sudo nano /etc/hosts
```

```sh
192.168.100.177 www.moodlecg.iesgn.org

```
#### Instalacion de moodle II

* Vamos a entrar desde nuestra máquina anfitriona a nuestro sitio web para instalar moodle. 

* Accedemos a www.moodlecg.iesgn.org

![m1.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m1.png)

* Seleccionamos el idioma.
* Dejamos las rutas como están porque son correctas.

![m2.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m2.png)

* Seleccionamos el gestor de base de datos mariaDB

![m3.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m3.png)

* Ponemos los datos de nuestra base de datos

![m4.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m4.png)

* Aceptamos los términos y condiciones

![m5.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m5.png)

* Vemos que todo está correcto y continuamos la instalación

![m6.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m6.png)

* Aquí nos advierte que somos vulnerables al no usar https y que cumplimos todos los requisitos

![m7.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m7.png)

* Continuar

![m8.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m8.png)


* Introducimos todos los datos del administrador de la cuenta de moodle y continuamos

![m9.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m9.png)

* Introducimos los datos de la página principal

![m10.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m10.png)

* Ya tendríamos instalada moodle en nuestro entormo cms local

![m11.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/m11.png)





