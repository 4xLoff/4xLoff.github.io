---
layout:   post
title:    "Write Up Pandora. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Linux,SNMP,PATH,RCE,LPF,Write-Up-Machine,Starting-Point,OSCP,eWPT] 
---
![list](/assets/img/pandora/Captura%20de%20pantalla%20(189).png){:.lead width="800" height="100" loading="lazy"}

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
nmap -p- --open --min-rate 5000 -n -vvv -Pn 10.10.11.136 -oA allports
```
El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}


***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que 

versiones y servicios que corren en los mismo de ahi podremos determinara posibles 

vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p22,8080 -sV -sC 10.10.11.136 -oN target
```

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}


![list](/assets/img/pandora/Arch-2022-05-24-19-05-31.png){:.lead width="800" height="100" loading="lazy"}


***

## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks.

Recopilación de información gracias a servicios de terceros.

Ispeccionamos la web.

Inpeccionamos las tecnologias de la ip ataves de la terminal o via  web esto lo 

hacemos con wapalizzer o whatweb.

![list](/assets/img/pandora/Arch-2022-05-24-19-08-26.png){:.lead width="800" height="100" loading="lazy"}

Pasamos mucho tiempo enumerando directorios y vhosts, pero sin éxito solo encontramos el 

dominio panda.htb. 

![list](/assets/img/pandora/Arch-2022-05-24-19-03-54.png){:.lead width="800" height="100" loading="lazy"}

Lo único que encontramos es un directorio `/assets` donde está habilitado el listado de 

directorios, la enumeración no salió tan bien, así que volvamos a Nmap y busquemos puertos 

UDP; anteriormente omitimos el escaneo UDP.

```bash
nmap -sU --top-ports 100 --open -T5 -n -v 10.10.11.136 -oA UDPports
```

Nunca omita los puertos UDP.
{:.note title="Attention"}




![list](/assets/img/pandora/Arch-2022-05-24-20-07-29.png){:.lead width="800" height="100" loading="lazy"}

```bash
nmap -p161 -sU -sV --min-rate=1000 -sC 10.10.11.136 -oN target
```

Solo el puerto 161 .
{:.note title="Attention"}


Snmpwalk sirve para consultar MIB de nuestro objetivo

![list](/assets/img/pandora/Arch-2022-05-24-20-44-00.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Nos desplazamos por la salida y notamos una combinación de usuario/contraseña.



Hemos obtenido las credenciales para el usuario `daniel`.Ahora intentemos conectar 

la máquina a través de SSH, pero desafortunadamente, no hay ningún indicador user.txt 

en el directorio de inicio de daniel.Pero hay otro usuario llamado `matt` que 

tiene acceso de lectura para acceder al archivo `user.txt` por lo que debemos pivotear. 


![list](/assets/img/pandora/Arch-2022-05-24-20-46-27.png){:.lead width="800" height="100" loading="lazy"}


***
## Explotacion

Siempre es bueno ejecutar `LimEnum` Y `pspy`, para que podamos enumerar más 

vulnerabilidades que permitan elevar nuestros privilegios al usuario "matt".

Vamos hacer un local port forwarding lo que significa que vamos a  atraer un puerto 

interno remoto a nuestro localhost en este caso el puerto 8080 a al puerto 80.

![list](/assets/img/pandora/Arch-2022-05-24-21-03-33.png){:.lead width="800" height="100" loading="lazy"}

```bash
ssh daniel@10.10.11.136 -L 80:127.0.0.1:80
```
{:.note}
Ya podemos acceder al servicio `pandora`.


![list](/assets/img/pandora/Arch-2022-05-24-21-08-17.png){:.lead width="800" height="100" loading="lazy"}

Para evitar fuzzear el localhost enviemos un:

```bash
curl localhost
```
{:.note}
Nos da la ruta `/pandora_console/`.



El servidor es vulnerable a SQLinjection.

![list](/assets/img/pandora/Arch-2022-05-24-21-35-28.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Comprobamos.


![list](/assets/img/pandora/Arch-2022-05-24-21-37-41.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Comprobamos el tamaño.


![list](/assets/img/pandora/Arch-2022-05-24-21-22-47.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Ahi mismo esta la version de pandora ya es cuestion de buscar un [exploit].


Si miramos el exploit esta usando una rutaque aplica una qwery para robar la cookie de 

session de tal manera que la vamos a usar y nos convertiremos en suporuesario.

`http://localhost/pandora_console/include/chart_generator.php?session_id=%27%20union%20SELECT%201,2,%27id_usuario|s:5:%22admin%22;%27%20as%20data%20--%20SgGO'`.

![list](/assets/img/pandora/Arch-2022-05-24-21-44-25.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
Somos admin.


Creamos un `archivo.php` para subir al servidor en la parte de file manager y lo subimos.

![list](/assets/img/pandora/Arch-2022-05-24-21-51-48.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Subimos y nos entabalmos una reverse shell desde la url.


![list](/assets/img/pandora/Arch-2022-05-24-22-47-16.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Nos otroga una shell como Matt.


![list](/assets/img/pandora/Arch-2022-05-24-22-32-01.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
La flag esta en el escritorio.


La shell es media rara asi que vamos a .ssh y creamps con ssh-keygen la id_rsa y id_rsa.pub

la copiamos y le ponemos en nombre de autorized_keys, le damos permisos 600 y nos la copiamos a

maquina atacante para conectarnos por ssh.

![list](/assets/img/pandora/Arch-2022-05-24-22-32-23.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
No olvidarse de darle permisos chmod 600 id_rsa.



Como vimos en daniel habia un backup que solo se puede habrir desde matt con `ltrace` 

podemos ver a bajo nivel el contenido con una ruta `tar -cvf /root/.backup/pandora-b` 

pero como no esta en la ruta absoluta lo que vamos hacer es un path hijacking,entonces 

en la ruta /tmp creamos un archivo que se llame tar y como contenido le metemos el comando 

`chmod u+s /bin/bash`,y ahregamos este archivoa al pat para que comiense la busque desde aki, 

desde `/tmp`.

![list](/assets/img/pandora/Arch-2022-05-24-22-45-55.png){:.lead width="800" height="100" loading="lazy"}

```shell
echo PATH
```
{:.note}
Vemos la ruta.

```shell
expot PATH:/tmp:$PATH
```
Hacemos bash -p ya somos root, buscamos la flag en el escritorio del root.
{:.note}


[exploit]: https://github.com/shyam0904a/Pandora_v7.0NG.742_exploit_unauthenticated

***

```shell
🎉 Felicitaciones ya has comprometido Pandora de HackTheBox 🎉
```
{:.centered}
***