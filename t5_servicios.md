# Necesidad de otros servicios

La mayoría de los CMS tienen la posibilidad de mandar correos electrónicos (por ejemplo para notificar una nueva versión, notificar un comentario,…)

### Instala un servidor de correo electrónico en tu servidor. debes configurar un servidor relay de correo, para ello en el fichero /etc/postfix/main.cf, debes poner la siguiente línea:

```sh
relayhost = babuino-smtp.gonzalonazareno.org
```

* Instalamos postfix

```sh
apt-get install mailutils postfix
```

* Elegimos 'Internet Site'

![correo1.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/correo1.png)

* Ponemos el nombre de nuestro dominio

![correo2.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/correo2.png)

* Ahora editamos el fichero de configuración y añadimos la línea que nos han propuesto en la tarea.

```sh
# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
# information on enabling SSL in the smtp client.

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = server
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = $myhostname, www.moodlecg.iesgn.org, server, localhost.localdomain, localhost
relayhost = babuino-smtp.gonzalonazareno.org
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.100.0/24
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all


```
* Instalamos bsd-mailx

```sh
apt instal bsd-mailx
```

* Reiniciamos postfix para que funcione correctamente

```sh
root@server:~# systemctl status postfix
● postfix.service - Postfix Mail Transport Agent
   Loaded: loaded (/lib/systemd/system/postfix.service; enabled; vendor preset: enabled)
   Active: active (exited) since Sun 2020-11-08 16:52:50 GMT; 7s ago
  Process: 27567 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
 Main PID: 27567 (code=exited, status=0/SUCCESS)

Nov 08 16:52:50 server systemd[1]: Starting Postfix Mail Transport Agent...
Nov 08 16:52:50 server systemd[1]: Started Postfix Mail Transport Agent.

```



### Configura alguno de los CMS para utilizar tu servidor de correo y realiza una prueba de funcionamiento.


He intentado utilizar el servidor de correos del IES pero no se puede puesto que no estamos en la misma red, habría que utilizar openvpn y entrar desde ahí para que funcione.


![intento.png](https://github.com/CeliaGMqrz/cms_install_debian/blob/main/capturas/intento.png)
