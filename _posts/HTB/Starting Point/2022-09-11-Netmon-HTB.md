---
layout:   post
title:    "Write Up Netmon."
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Windows,FTP,CVE,RCE,Weak-Authentication,Write-Up-Machine,Starting-Point,eJPT,eWPT,OSCP] 
---
![list](/assets/img/netmon/Captura de pantalla (142).png){:.lead width="800" height="100" loading="lazy"}

***
<!--more-->

1. this ordered seed list will be replaced by the toc
{:toc}

***

## Reconocimiento

Fase inicial donde recolectamos toda la información posible del objetivo, utilizando diferentes técnicas como:

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
nmap -p- --open --min-rate 5000 -sS -n -vvv -Pn 10.10.10.152 -oA allports
```

![list](/assets/img/netmon/Kali-2022-09-09-15-00-11.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}
***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.

```bash
nmap -p21,135,139,445,5985,47001,49664,49665,49666,49667,49668,49669 -sV -sC 10.10.10.152 -oN target
```

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***
## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks y recopilación de información gracias a servicios de terceros, aunque tenemos muchos puertos abiertos niguno de ellos representa una vulnerabilidad exectuando el pueto 21 FTP ademas de el puento80 que es Web.

```bash
ftp 10.10.10.52
```
Husmeando por los direcctorios en el escritorio nos encontramos con la flag de user.
{:.note}



Enumeramos las tecnologias que usa la web con whatweb o wapalizer.

![list](/assets/img/netmon/Kali-2022-09-09-15-18-56.png){:.lead width="800" height="100" loading="lazy"}


Tambien si nos vamos a ala web vemos que temenos la capacidad de loogearnos lo cual podemos probar usar contraseñas por defecto bucando por internet.

Si busacmo en internet por path traversal windows presseler o o buscando manualmente en el ftp daremos con el archivo de configuracion de PRTG y tambien la copia de seguridadcon el comando wget nos decargamos los dos archivos a la maquina atacante para analizar dichos documentos.

```bash
curlftpfs ftp://10.10.10.152 /mnt/montura
```
Esto es para movilizarnos con fluides por el ftp pero no esnesesario. 
{:.note}



Para movilizanos con rapides por los direcctorios de FTP podiamos crear una montura con certutils.

```bash
diff Configuration.old Configuration.old.bak -y | less
```

{:.note}
El parametro -y es parq que en pantalla aparescan los dos documetos y sea mas facil su analisis, encontramos un ususrio y contraseña que usaremos en la pagina web para loogearnos.


![list](/assets/img/netmon/Kali-2022-09-09-16-20-57.png){:.lead width="800" height="100" loading="lazy"}
User=prtgadmin pass=PrTg@dmin2018
{:.note}

![list](/assets/img/netmon/Kali-2022-09-09-16-21-48.png){:.lead width="800" height="100" loading="lazy"}

Si buscamos en internet algun exploit relacionado con PRTG o viendo los archivos de la licencia de PRTG vemos que has una ruta a la cual hacen refercia que es `/myaccount.htm?tabid=2` la cual si la pegamos des pues de la ip uan ves loogeados no rdirigue a una seccion donde se pueden crea notificaciones.

![list](/assets/img/netmon/Kali-2022-09-09-16-23-12.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/netmon/Kali-2022-09-09-16-37-44.png){:.lead width="800" height="100" loading="lazy"}

En el ftp esta en codigo html lo cual podemos decodearlo en la terminal o via web en [codebeautify].
{:.note}

[codebeautify]: https://codebeautify.org/html-decode-string

***
## Explotacion y Escalacion de privilegios

Tenemos varias formas de explotar obtener la flag root pero vamos a ver tres.

### Primera forma

es usar el exploit de github el cual nos dice que nesesitamos la cookie de seccion la cual interceptamos con burpsuite.

![list](/assets/img/netmon/Kali-2022-09-09-17-33-04.png){:.lead width="800" height="100" loading="lazy"}
Interceptar la cookie.
{:.note}

![list](/assets/img/netmon/Kali-2022-09-09-17-34-35.png){:.lead width="800" height="100" loading="lazy"}
El exploit lo que hace es que crea o injecta un  usuario y pass para el usuario root.
{:.note}


Luego con crackmaapexec nos loogeamos y obtenemos la flag de root.

```bash
crackmaapexec smb 10.10.10.152 -u "pentest" -p "P3nTest!"
```
***
### Segunda forma

Nos debemos crear una noticicacion en la cual podemos inyectar un usuario y pass para usar crackmapexex o ssh ,o una reverse shell.

![list](/assets/img/netmon/Kali-2022-09-09-16-49-48.png){:.lead width="800" height="100" loading="lazy"}
Creamos una notificacion llamada test.
{:.note}


En la parte mas baja en la seccion de remote execution code existe la posibilidad de insetar una lineas de comando para hacer lo antes mecionado de la siguiente manera.

```bash
C:\\User\\Public\\Texter; net user test test123 add/ 
```
Ya creado usamos crackmapexec. 
{:.note}

### Tercera forma

Es lo mismo que el paso anterio pero en ves crear usuario y contraseña copiamos la flags de root al directio FTP.

![list](/assets/img/netmon/Kali-2022-09-09-17-24-33.png){:.lead width="800" height="100" loading="lazy"}

```bash
tester.txt Copy-Item -Path "C:\Users\Administrator\Desktop\root.txt" -Destination "C:\Users\Public\texter.txt" -Recurse
```
Ya seria diriguirse a la  ruta donde guardamos el recurso en el FTP. 
{:.note}


***
```bash
🎉 Felicitaciones ya has comprometido Netmon de HackTheBox 🎉
```
{:.centered}
***

Back to [Beginner Track](2022-09-12-Beginner-Track.md){:.heading.flip-title}
{:.read-more}

