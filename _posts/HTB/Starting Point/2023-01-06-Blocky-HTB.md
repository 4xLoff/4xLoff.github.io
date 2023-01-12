---
layout:   post
title:    "Write Up Blocky. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Medium,Linux,java,Wordpress,Sudoers,Write-Up-Machine,Starting-Point,eWPT,eJPT] 
---
![list](/assets/img/blocky/Captura%20de%20pantalla%20(287).png){:.lead width="800" height="100" loading="lazy"}

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
nmap -p- --open --min-rate 5000 -n -vvv -Pn 10.10.10.37 -oA allports
```
El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}


![list](/assets/img/blocky/Parrot-2022-12-20-14-39-23.png){:.lead width="800" height="100" loading="lazy"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p80 -sV -sC 10.10.10.37 -oN target
```
![list](/assets/img/blocky/Parrot-2022-12-20-14-42-12.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***

## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks y recopilación de información gracias a servicios de terceros.e ispeccionamos la web, ademas de las tecnologias de la ip ataves de la terminal o via  web esto lo hacemos con wapalizzer o whatweb, como encontramos version vulnerable del srvicio ftp con serasploit tambien vamos a investigar ese vector .

![list](/assets/img/blocky/Parrot-2022-12-20-14-47-48.png){:.lead width="800" height="100" loading="lazy"}

Whatweb nos  esta aplicando un reirerct al dominio blocky.htb por lo que lo ingresamos al `/etc/hosts`.

![list](/assets/img/blocky/Parrot-2022-12-20-14-44-11.png){:.lead width="800" height="100" loading="lazy"}

Tambien podemos fuzzear los directorioso con nmap antes de usar otros fuzzers mas completos para ver que informacion nos entrega.

![list](/assets/img/blocky/Parrot-2022-12-20-14-52-42.png){:.lead width="800" height="100" loading="lazy"}

Por si cacaso fuzeamos con wfuuzz y nos arrogo lo siguiente.

![list](/assets/img/blocky/Parrot-2022-12-20-15-11-03.png){:.lead width="800" height="100" loading="lazy"}

En el directorio plugins ahi un archivo que se puede descragar que se llama `BlockyCore.jar` el caul nos vamos a decargas y lo investigaremos para ver que tipo de informacion le podemos sacar.

![list](/assets/img/blocky/Parrot-2022-12-20-15-22-10.png){:.lead width="800" height="100" loading="lazy"}

Logramos opbtener la credenciales para el usuario `notch:8YsqfCTnvxAUeduzjNSXe22`.
{:.note}

Ya solo nos conectamos atraves de SSH.

![list](/assets/img/blocky/Parrot-2022-12-20-15-37-18.png){:.lead width="800" height="100" loading="lazy"}


Ya podemos buscar la flag de bajos privileguios en el escritorip del usuario `notch`. 
{:.note}

***
## Explotacion

Siempre es bueno ejecutar `LimEnum` Y `pspy` para monitorizar y ver los posibles vectores para escalar de privilegios, ademas hasta que se ejecute los anteriores podemos buscar los permisos con el comado find o tareas crond o tareas que esperen ejecucion.

para escalar privileguios yo me percate de dos formas y esta cuando hacemos id estamos en dos grupos vulnerables el [lxd] y el grupo sudo este ultimo es que vamos a explotar , como somos sudo lo que resta es hacer sudo su y la clave `8YsqfCTnvxAUeduzjNSXe22` ya nos convertiriamos en root.

![list](/assets/img/blocky/Parrot-2022-12-20-15-39-45.png){:.lead width="800" height="100" loading="lazy"}

[lxd]:(https://www.exploit-db.com/exploits/46978)


{:.note}
Lo ultimo es buscar la flag de superusuario en el escritorio de root.

***

```shell
🎉 Felicitaciones ya has comprometido Blocky de HackTheBox 🎉
```
{:.centered}
***

Back to [Beginner Track](2022-09-12-Beginner-Track.md){:.heading.flip-title}
{:.read-more}