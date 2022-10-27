---
layout:   post
title:    "Write Up Ambassador. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Medium,Linux,Grafana,Token,SSH,Write-Up-Machine,Starting-Point] 
---
![list](/assets/img/abassador/Captura%20de%20pantalla%20(185).png){:.lead width="800" height="100" loading="lazy"}

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
nmap -p- --open --min-rate 5000 -n -vvv -Pn 10.10.11.183 -oA allports
```
El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}


![list](/assets/img/abassador/Kali-2022-10-05-19-04-48.png){:.lead width="800" height="100" loading="lazy"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que 

versiones y servicios que corren en los mismo de ahi podremos determinara posibles 

vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p22,8080 -sV -sC 10.10.11.183 -oN target
```
![list](/assets/img/abassador/Kali-2022-10-05-19-07-49.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***

## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks.

Recopilación de información gracias a servicios de terceros.

Ispeccionamos la web.

Inpeccionamos las tecnologias de la ip ataves de la terminal o via  web esto lo 

hacemos con wapalizzer o whatweb.

Fuzzeamos la web y los puertos 80 y 3000 para encontrar directorios.

![list](/assets/img/abassador/Kali-2022-10-05-19-25-14.png){:.lead width="800" height="100" loading="lazy"}
En el 80 no encontramos nada interesante. 
{:.note}



Miramos el puerto 3000 podemos ver lo siguiente.

![list](/assets/img/abassador/Kali-2022-10-05-19-24-21.png){:.lead width="800" height="100" loading="lazy"}
Con `curl http://10.10.11.183:3000/login | grep "Grafana v"` podemos ver la version. 
{:.note}


Buscando exploits en la web o con searsploit en contramos una vulneravilidad de 

Directory Traversal and Arbitrary File para [Grafana], para leer archivos.

[Grafana]:https://vk9-sec.com/grafana-8-3-0-directory-traversal-and-arbitrary-file-read-cve-2021-43798/

Usamos el exploit que encontramos para enumerar los usuarios de la victima.

![list](/assets/img/abassador/Kali-2022-10-05-19-16-09.png){:.lead width="800" height="100" loading="lazy"}

Con `cat credential.txt | grep "sh$"` filtramos por los usuarios validos . 
{:.note}



También podemos descargar la db de grafana para inspeccionarla.

![list](/assets/img/abassador/Kali-2022-10-05-19-23-02.png){:.lead width="800" height="100" loading="lazy"}

```bash
curl -s --path-as-is http://10.10.11.183:3000/public/plugins/alertlist/../../../../../../../../var/lib/grafana/grafana.db -o grafana.db
```


Husmenado en la base de datos nos encontramos una credecial `dontStandSoCloseToMe63221!`.
{:.note}


En este punto ya es conectarnos a la base de datos del servidor hya tenemos un nombre de

usuario que es developer y la pass.

![list](/assets/img/abassador/Kali-2022-10-05-19-38-25.png){:.lead width="800" height="100" loading="lazy"}

Volvemos a husmear y encontramos nuevas credenciales que estan en base64 por lo cual 

debemos decodearlas ya sea via web o terminal como quieras.

![list](/assets/img/abassador/Kali-2022-10-05-19-39-23.png){:.lead width="800" height="100" loading="lazy"}
Las nuevas crededenciales son: `developer:YW5FbmdsaXNoTWFuSW5OZXdZb3JrMDI3NDY4Cg==`.
{:.note}



Nos conectamos via ssh a ssh developer@10.10.11.183

![list](/assets/img/abassador/Kali-2022-10-05-19-42-45.png){:.lead width="800" height="100" loading="lazy"}
Como siempre la flag de user esta en el escritorio de developer.
{:.note}

***
## Explotacion

Siempre es bueno ejecutar `LimEnum` Y `pspy` para monitorizar y ver los posibles 

vectores para escalar de privilegios, ademas hasta que se ejecute los anteriores 

podemos buscar los permisos con el comado find con el siguite comado.

Buscando entre permisos root se encontró una carpeta propiedad de root con acceso de 

lectura para el desarrollador y contiene un repo .git, en cual miraremos la diferencias entre los commits 

![list](/assets/img/abassador/Kali-2022-10-05-21-25-52.png){:.lead width="800" height="100" loading="lazy"}
Encontramos un token `bb03b43b-1d81-d62b-24b5-39540ee469b5`.
{:.note}




En la búsqueda en línea se han encontrado los siguientes recursos relativos a la explotación del consul [API].

[API]: https://www.infosecmatter.com/metasploit-module-library/?mm=exploit/multi/misc/consul_service_exec

Utilizamos mfscosole para conectarnos al servidor como root con payload que inyecta el acl_token.

![list](/assets/img/abassador/Kali-2022-10-05-22-31-29.png){:.lead width="800" height="100" loading="lazy"}

Es inportalte que en el msfconsole se agregue el campo ACL_token `bb03b43b-1d81-d62b-24b5-39540ee469b5`.
{:.note}

Ya lo ultimo es buscar la flag de superusuario en el escritorio de root.

***

```shell
🎉 Felicitaciones ya has comprometido Ambassador de HackTheBox 🎉
```
{:.centered}
***