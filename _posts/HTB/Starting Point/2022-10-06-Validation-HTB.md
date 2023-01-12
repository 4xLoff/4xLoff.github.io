---
layout:   post
title:    "Write Up Validation. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Linux,SQLi,RCE,Write-Up-Machine,Starting-Point,eJPT,eWPT] 
---
![list](/assets/img/validation/Captura%20de%20pantalla%20(295).png){:.lead width="800" height="100" loading="lazy"}

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
nmap -p- --open --min-rate 5000-sS -n -vvv -Pn 10.10.11.116 -oA allports
```


El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}


![list](/assets/img/validation/A-2022-12-14-16-03-27.png){:.lead width="800" height="100" loading="lazy"}


***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p22,80,4566,8080 -sV -sC 10.10.11.116 -oN target
```


El paso anterior no es nesesario ser (root).
{:.note title="Attention"}
***

## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks y recopilación de información gracias a servicios de terceros.e ispeccionamos la web, ademas de las tecnologias de la ip ataves de la terminal o via  web esto lo hacemos con wapalizzer o whatweb.

![list](/assets/img/validation/A-2022-12-14-16-05-09.png){:.lead width="800" height="100" loading="lazy"}

Podemos primero probar otrsos posibles ataques como HTMLinjection, XSS ,SQLinjection, pero como no estamos autenticadoa mejor vamos a pasar a burpsuite para hacer las consultas de una manera mas facil y por eso vamos a interceptar las peticiones y hacer las pruebas basica de injeccion SQL.

![list](/assets/img/validation/A-2022-12-14-16-21-28.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Pese que no se represente en la respuesta si se represente en el render y podemos ver la database que en este caso es `registration`.

![list](/assets/img/validation/A-2022-12-14-16-22-52.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Vemos las version de la base de datos.

![list](/assets/img/validation/A-2022-12-14-16-33-08.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Vemos las tablas.

![list](/assets/img/validation/A-2022-12-14-16-36-31.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Vemos un usuario y contraseña.

Aprovechado que ya estamos aki tambien vam,os a tratar de mandarnos una reverse-shell por lo quem vamos a injectarle un codigo php que puede ejecutar comados.

![list](/assets/img/validation/A-2022-12-14-16-44-42.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/validation/A-2022-12-15-13-11-32.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Debemos estar a ala ecucha con netcat en el puerto 443.

Ya podemos buscar la flag de bajos privileguios en el escritorip del usuario htb. 
{:.note}

***
## Explotacion

Siempre es bueno ejecutar `LimEnum` Y `pspy`, para que podamos enumerar más vulnerabilidades que permitan elevar nuestros privilegios al usuario "root".

Husmeando por los directorios en `/var/www/html`  hay un documento config.php que tiene unas credenciales `uhc-9qual-global-pw` que vamos a probar con el usuario root haciendo el comado su root y pegando uhc-9qual-global-pw.

![list](/assets/img/validation/A-2022-12-15-13-21-18.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/validation/A-2022-12-15-13-43-30.png){:.lead width="800" height="100" loading="lazy"}

Ya podemos, buscar la flag en el escritorio del root.
{:.note}

***

```shell
🎉 Felicitaciones ya has comprometido Validation de HackTheBox 🎉
```
{:.centered}
***

Back to [Beginner Track](2022-09-12-Beginner-Track.md){:.heading.flip-title}
{:.read-more}