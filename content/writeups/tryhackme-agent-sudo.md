---
title: "TryHackMe | Agent Sudo"
published: 2025/05/30
slug: "tryhackme-agent-sudo"
---

<img src="/writeups/tryhackme-agent-sudo/thumbnail.png" width=200px heigh=200px/>

<https://tryhackme.com/room/agentsudoctf>

## Escaneo

Escaneamos todos los puertos TCP de la maquina objetivo con Nmap.

```bash
sudo nmap -n -Pn -T4 10.10.195.209 -oN nmap.txt
```

```text
sudo nmap -n -Pn -T4 10.10.195.209 -oN nmap.txt

# Nmap 7.95 scan initiated Fri May 30 16:11:54 2025 as: /usr/lib/nmap/nmap -p- -n -Pn -T4 -oN nmap.txt 10.10.195.209
Nmap scan report for 10.10.195.209
Host is up (0.27s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

Obtenemos información de los puertos abiertos.

```bash
sudo nmap -sCV -n -Pn -T4 10.10.195.209 -oN nmap-ports.txt
```

```text
sudo nmap -sCV -n -Pn -T4 10.10.195.209 -oN nmap-ports.txt

# Nmap 7.95 scan initiated Fri May 30 16:25:00 2025 as: /usr/lib/nmap/nmap -sCV -p 21,22,80 -n -Pn -T4 -oN nmap-ports.txt 10.10.195.209
Nmap scan report for 10.10.195.209
Host is up (0.27s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Annoucement
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
```

> Para agilizar el proceso de escaneo y enumeración, primero encontramos puertos abiertos y luego los escaneamos en busca de banners, versiones y otra información util.

## Enumeración de servicios

### TCP/21

No es posible realizar un inicio de sesión anónimo en el servicio FTP. Por lo que tendremos que revisitar este puerto mas adelante.

### TCP/80

Se nos da la bienvenida con un mensaje críptico.

![image-1](/writeups/tryhackme-agent-sudo/image-1.png)

"user-agent" se refiere a la cabecera HTTP "User-Agent", por lo que debemos de cambiarla para poder acceder. ¿Pero que "código nombre" debemos utilizar? Puesto que el mensaje fue firmado por Agent R, podemos intuir que nuestro código nombre corresponde a alguna de las letras del abecedario.

Creamos una lista de palabras con todas las letras del abecedario en mayúscula haciendo uso de **TTPassGen**.

```bash
ttpassgen -r "[?u]{1:1}" wordlist.txt
```

```text [wordlist.txt]
A
B
C
<SNIP>
Y
Z
```

Enviamos solicitudes con **ffuf**. Puesto que la lista de palabras es corta, y no sabemos exactamente que esperar de una respuesta adecuada, podemos optar por no filtrar las respuestas y examinarlas manualmente. Sin embargo, también podemos suponer que una respuesta adecuada contendrá un código de respuesta diferente a 200 o tendrá un tamaño diferente al usual (218 bytes).

Para propósitos educativos, elegiremos seguir las redirecciones y filtrar las respuestas basadas en su tamaño.

```bash
ffuf -u http://10.10.195.209 -H "User-Agent: FUZZ" -w wordlist.txt -c -r -fs 218
```

> ¿Filtrar por código de respuesta o por tamaño? Filtrar por código de respuesta nos puede llevar a omitir respuestas no estándares, por lo que seguiremos la redirecciones y filtraremos por tamaño de respuesta.

```text
ffuf -u http://10.10.195.209 -H "User-Agent: FUZZ" -w wordlist.txt -c -r -fs 218

<SNIP>
R                       [Status: 200, Size: 310, Words: 31, Lines: 19, Duration: 267ms]
C                       [Status: 200, Size: 177, Words: 27, Lines: 8, Duration: 267ms]
```

> La letra C originalmente nos redirige a otra pagina, mientras que la letra R nos muestra otro mensaje. Si hubiéramos filtrado todas las respuestas con código 200, hubiéramos omitido esta ultima respuesta.

Al enviar una petición con el User-Agent "R", obtenemos una respuesta ligeramente diferente a la original (pero sin importancia). La letra "C" nos revela una respuesta completamente diferente.

```bash
curl -sL -H "User-Agent: C" http://10.10.195.209/
```

```text
curl -sL -H "User-Agent: C" http://10.10.195.209/

Attention chris, <br><br>

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! <br><br>

From,<br>
Agent R
```

Descubrimos el nombre de usuario "chris" y que su contraseña es débil.

Si intentamos enumerar directorios y paginas web, tales como formularios de inicio de sesión, no obtendremos resultados.

## Explotación

### SSH

Al no encontrar un vector de explotación evidente, usaremos **hydra** para obtener la contraseña del servicio SSH mediante fuerza bruta.

Iniciar sesión en este servicio corresponde al camino hacia el éxito mas directo, por tanto tiene sentido probar un ataque de fuerza bruta primero con este servicio. Pero en un entorno real este servicio estará fuertemente protegido y monitorizado, por lo que deberíamos de probar en otros lugares primero.

Finalmente, este ataque falla en obtener una contraseña valida en un intervalo de tiempo aceptable.

> Nunca debes realizar un ataque de fuerza bruta, a menos que sea el único camino disponible o hayan grandes indicios de que este es el camino correcto.

### FTP

Al fallar el anterior ataque, esta vez lo intentaremos en el servicio FTP.

> Si hay indicios de que una contraseña es débil, vale la pena usar listas de palabras conocidas como rockyou.txt (Solo en CTFs).

```bash
hydra -l "chris" -P /usr/share/wordlists/rockyou.txt -f -vV 10.10.195.209 ftp
```

```text
hydra -l "chris" -P /usr/share/wordlists/rockyou.txt -f -vV 10.10.195.209 ftp

<SNIP>
[ATTEMPT] target 10.10.195.209 - login "chris" - pass "freedom" - 256 of 14344399 [child 9] (0/0)
[21][ftp] host: 10.10.195.209   login: chris   password: crystal
[STATUS] attack finished for 10.10.195.209 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-30 20:05:43
```

Ingresamos al servidor FTP con las anteriores credenciales y listamos los archivos.

```text
ftp> ls
229 Entering Extended Passive Mode (|||6987|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
```

Descargamos los archivos de forma recursiva.

```text
ftp> prompt
Interactive mode off.
ftp> mget .
local: To_agentJ.txt remote: To_agentJ.txt
<SNIP>
local: cute-alien.jpg remote: cute-alien.jpg
<SNIP>
local: cutie.png remote: cutie.png
<SNIP>
```

> Usamos el comando "promt" para evitar tener que confirmar manualmente la descarga de cada archivo.

A simple vista, no hay nada fuera de lo usual en las imágenes descargadas. El archivo de texto contiene el siguiente mensaje.

```text [To_agentJ.txt]
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

### Esteganografía

Una de las imágenes contiene la contraseña de Agent R. Debemos realizar estegoanálisis en busca de un mensaje escondido. Podemos realizar este proceso con muchas herramientas, pero en CTFs usualmente podremos obtener información oculta haciendo uso de *exiftool*, *strings*, *binwalk* y *steghide* (No esperes hacer uso de estas herramientas en pruebas de penetración reales).

*exiftool* no revela nada importante, pero **strings** si lo hace.

```bash
strings cutie.png | head -n 20 && echo "....." && strings cutie.png | tail -n 20
```

```text
strings cutie.png | head -n 20 && echo "....." && strings cutie.png | tail -n 20

<SNIP>
To_agentR.txt
W\_z#
2a>=
To_agentR.txt
EwwT
```

Descubrimos que la imagen contiene un archivo de texto oculto. Para extraerlo hacemos uso de **binwalk**.

```bash
binwalk -e cutie.png
```

```text
binwalk -e cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt

WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

Los archivos fueron extraídos en una nueva carpeta. Dentro de esta encontramos un archivo zip, sin embargo *unzip* falla en extraer este archivo, por lo que debemos de usar **7zip**.

```bash
7zip x 8702.zip
```

### Crackeo de zip

El archivo esta protegido por una contraseña. Para *crackearla* haremos uso de **zip2john**.

```bash
zip2john 8702.zip > zip2john.txt
```

*Crackeamos* la contraseña con **john**.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt zip2john.txt
```

```text
john --wordlist=/usr/share/wordlists/rockyou.txt zip2john.txt

<SNIP>
alien            (8702.zip/To_agentR.txt)
1g 0:00:00:00 DONE (2025-05-30 19:47) 1.351g/s 44281p/s 44281c/s 44281C/s christal..eatme1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

El archivo descomprimido contiene el siguiente mensaje.

```text [_cutie.png.extracted/To_agentR.txt]
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

Al principio esto no parece sernos de utilidad, pero la cadena de texto "QXJlYTUx" puede estar codificada. Ingresamos esta cadena de texto en **CyberChef**. La pagina automáticamente reconoce que esta cadena de texto esta codificada en Base64. Al decodificarla obtenemos la frase "Area51".

¿Obtuvimos la contraseña de SSH? Resulta no ser valida. Si esta imagen contiene un archivo escondido, probablemente la otra imagen también.

**strings** revela `12345`

## Escalada de privilegios

Descripción

## Persistencia

Descripción

## Post-Root

Descripción

## Resumen ejecutivo

Descripción

## Remediación

1. Evita el uso de contraseñas débiles.

## Conclusiones

Descripción
