---
layout:   post
title:    "Write Up Squashed. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Linux,NFS,.Xauthority,Write-Up-Machine,Starting-Point,OSCP] 
---
![list](/assets/img/squashed/Captura%20de%20pantalla%20(279).png){:.lead width="800" height="100" loading="lazy"}

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
nmap -p- --open --min-rate 5000-sS -n -vvv -Pn 10.10.11.191 -oA allports
```


El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}


![list](/assets/img/squashed/A-2022-12-08-10-40-10.png){:.lead width="800" height="100" loading="lazy"}


***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p22,80,111,2049,36559,41923,45703,54571 -sV -sC 10.10.11.191 -oN target
```


![list](/assets/img/squashed/A-2022-12-08-10-41-07.png){:.lead width="800" height="100" loading="lazy"}


El paso anterior no es nesesario ser (root).
{:.note title="Attention"}
***

## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks y recopilación de información gracias a servicios de terceros.e ispeccionamos la web, ademas de las tecnologias de la ip ataves de la terminal o via  web esto lo hacemos con wapalizzer o whatweb. 

![list](/assets/img/squashed/A-2022-12-08-10-48-12.png){:.lead width="800" height="100" loading="lazy"}

Ademas esta expuesto el puerto 80 lo tratamos de fuzzear pero tampoco conseguimos nada lo cual me hace pensar que la intrusion de esta maquina no es por los puertos bajos asi que vamosa investigar sobre los demas puertos.

![list](/assets/img/squashed/A-2022-12-08-10-46-40.png){:.lead width="800" height="100" loading="lazy"}

Entoces vamos a invetigar el puerto 2049 en [HACKTRIKS].

[HACKTRIKS]:(https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)

Tal parece que con el comando `showmount 10.10.11.191` podemos ver recursos de la maquina victima que debemos hacer monturas para cada una de ellas en nuestro equipo local.

```shell
mount -t nfs 10.10.11.191:/home/ross /mnt/ross -o nolock
```

```shell
mount -t nfs 10.10.11.191:/var/www/html /mnt/server -o nolock
```


![list](/assets/img/squashed/A-2022-12-08-11-05-42.png){:.lead width="800" height="100" loading="lazy"}

Husmenado por las  monturas encontramos algunos archivos tales como un `Password.kdbx` en la montura de `ross`, pero la de server no tenemos permisos.

![list](/assets/img/squashed/A-2022-12-08-11-07-50.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/squashed/A-2022-12-08-11-08-04.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Esto pasa porque el identificador es 2017, por lo que para arreglar esto debemos agrgar un usuario con ese identificado de la siguiente manera:

```shell
useradd test
```

```shell
usermod -u 2017 test
```
Investigando dentro de la montura de server se ve que donde se aloja la paguna web y como vimos con whatweb la pagina usa php como lenguaje, por lo que vamos a crear un archivo `cmd.php` con codigo para que nos interprete una cmd desde la web y otrogarnos una reverse-shell con la tipica de [pentestmonkey].

![list](/assets/img/squashed/A-2022-12-08-11-24-44.png){:.lead width="800" height="100" loading="lazy"}


[pentestmonkey](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)

![list](/assets/img/squashed/A-2022-12-08-11-20-47.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Debemos estar a ala ecucha con netcat en el puerto 443 y buscar la flag de bajor privileguios en el escritorio del usuario alex.

***
## Explotacion

Siempre es bueno ejecutar `LimEnum` Y `pspy`, para que podamos enumerar más vulnerabilidades que permitan elevar nuestros privilegios al usuario "root".

Haciendo un porco de memoria teniamos ala monrura de ros que pudimo ingresar pero tampoco teniamos accrso atodo los archivos por lo que que debemos hacer lo mismo de ante de crar un usuario con identificador para leer el archivo `.xauthority` esto sirve para poder ver la pantalla de usuario roos.

![list](/assets/img/squashed/A-2022-12-08-11-09-33.png){:.lead width="800" height="100" loading="lazy"}


```shell
useradd test2
```

```shell
usermod -u 1001 test2
```

Con el comado w vemos el displey seteado a 0 y es judto lo que nesesitamos para verificar la conexion.

![list](/assets/img/squashed/A-2022-12-08-11-36-01.png){:.lead width="800" height="100" loading="lazy"}

Para conseguir esto el `.xauthority` que esta en la montura de ross debemos subirlo a la maquina victima para que el usuario alex tenga el mismo archivo y que se utilizar el comando anterior.
{:.note}


Luego de eso debemos de tomar una captura de la pantalla con el siguiente comando.


```shell
xwd -root -screen -silent -display 10.10.11.191 > screenshot.xwd
```

Con `convert screenshot.xwd screenshot.png` lo convertimos a png y si abrimos eso con cualquier gestor de imagenes vemos que es la contraseña `cah$mei7rai9A`. 
{:.note}



![list](/assets/img/squashed/A-2022-12-08-11-36-01.png){:.lead width="800" height="100" loading="lazy"}

Y con la contraseña de root que es nos podemos loogear y buscar la flag en el ecritorio del mismo.
{:.note}

***

```shell
🎉 Felicitaciones ya has comprometido Squashed. de HackTheBox 🎉
```
{:.centered}
***

Back to [Beginner Track](2022-09-12-Beginner-Track.md){:.heading.flip-title}
{:.read-more}