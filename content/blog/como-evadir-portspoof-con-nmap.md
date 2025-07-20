---
title: "Como evadir \"portspoof\" con Nmap"
description: "Aprende sobre \"portspoof\", una herramienta para camuflar puertos cerrados como activos, y como evadirla durante el escaneo de un host con Nmap."
published: 2025/07/18
slug: "como-evadir-portspoof-con-nmap"
---

![thumbnail](/blog/como-evadir-portspoof-con-nmap/thumbnail.png)

Durante la realización del desafió "Cheese CTF" de TryHackMe me tope con un resultado inusual de Nmap luego de escanear los puertos TCP de la maquina objetivo.

![nmap-scan](/blog/como-evadir-portspoof-con-nmap/nmap-scan.png)

¿Acaso todos los puertos se encuentran abiertos? Nuestra intuición nos dice que esto no puede ser posible.

Efectivamente, hay medidas de seguridad implementadas en el servidor web que camuflan los puertos cerrados como abiertos, lo que imposibilita a un atacante (como nosotros) de distinguir los puertos que realmente están abiertos de los que no lo están, pero pretenden estarlos. Si hubiéramos escaneado todos los 65535 puertos del servidor web, los resultados serian abrumadores y completamente inútiles desde nuestra perspectiva como atacante.

Y un escaneo de versiones o una ejecución de scripts del Nmap Scripting Engine seria aun mas catastrófico, llegando a tardar inclusive horas y presentando resultados completamente confusos y sin valor alguno.

![nmap-version-scan](/blog/como-evadir-portspoof-con-nmap/nmap-version-scan.png)

> Este escaneo tomo 33 minutos en completarse y el resultado debe ser examinado cuidadosamente para distinguir los posibles puertos abiertos de los que realmente no lo están.

Afortunadamente para *rootear* esta maquina tan solo necesitamos del puerto 22 (SSH) y 80 (HTTP), ademas de conocimiento de filtros PHP para transformar un LFI en un RCE.

Luego de completar este desafió, me puse en la tarea de averiguar que estaba sucediendo durante el escaneo de puertos y me tope con la herramienta que producía estos resultados falsos: [portspoof](https://drk1wi.github.io/portspoof/).

## Introducción a portspoof

Según su autor, esta herramienta permite mejorar la seguridad de los sistemas a traves de una serie de técnicas de camuflaje. Los resultados de los escaneos de puertos de un atacante se verán completamente destrozados y carentes de sentido, ya que ``portspoof`` responderá con un paquete ``SYN/ACK`` a las conexiones TCP en los puertos configurados (simulando que el puerto esta abierto), ademas de que responderá con *banners* validos generados dinámicamente a partir de una base de datos, lo que dificulta a un atacante el identificar si el puerto realmente contiene un servicio activo o no, y que información puede extraer de este servicio.

![wireshark](/blog/como-evadir-portspoof-con-nmap/wireshark.png)

> Los puertos 1 y 3 no están realmente abiertos, pero portspoof responde a nuestros paquetes SYN con un SYN/ACK.

Ya entrando en detalles, para que esta herramienta funcione correctamente es necesario hacer uso de ``iptables`` para crear una nueva regla en la tabla NAT que redirija el trafico TCP del puerto que queremos "camuflar" al puerto en el que opera el demonio de ``portspoof`` (por defecto es el puerto TCP 4444), excluyendo los puertos que realmente estan abiertos y queremos que sean accesibles. Todo esto lo podemos automatizar con un script de bash (tomado de <https://www.vicarius.io/vsociety/posts/research-evading-portspoof-solution>).

```bash
#!/bin/bash
spoofPorts="1:19 23:24 26:52 54:79 81:109 112:122 124:442 444:464 466:586 588:891 893:2048 2050:8079 8081:32800 32801:65535"
for prange in ${spoofPorts}; do
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport ${prange} -j REDIRECT --to-ports 4444
done
```

### N0Pspoof: ¿Una herramienta para evadir portspoof?

Durante mi investigación de ``portspoof`` encontré una herramienta titulada como ``N0Pspoof`` diseñada para evadir ``portspoof`` y creada por el mismo autor del anterior articulo. Pero te aseguro que esta herramienta no funcionó en evadir ``portspoof`` durante el desafió "Cheese CTF", ademas de que por su naturaleza es extremadamente lenta al probar la disponibilidad de una gran cantidad de puertos y es infinitamente menos versátil que ``nmap``.

![n0pspoof](/blog/como-evadir-portspoof-con-nmap/n0pspoof.png)

> ¿Estas seguro de eso N0Pspoof?

### Evasión manual (pero tediosa)

Una vez el trafico de un puerto "camuflado" sea redirigido a ``portspoof``, este responderá con un paquete ``SYN/ACK`` para indicar que el puerto se encuentra falsamente abierto y generara un *banner* falso basado en el numero de puerto en cuestión. Finalmente cerrara la conexión. De esta forma, tanto escáneres automáticos como Nmap y pruebas manuales con ``telnet`` o ``netcat`` darán resultados poco fiables en cuanto a si el puerto esta abierto o no, y que servicio se encuentra activo dentro de este.

La única forma de sortear esta herramienta es interactuando manualmente con cada puerto con un cliente adecuado o con herramientas como ``telnet`` y ``netcat``, y verificar si el puerto responde como debería según el servicio que (probablemente) se encuentre alojado en este. Por su puesto, hacer esto con cada puerto es tedioso, más aun si queremos probar puertos poco comunes.

## Evadir portspoof con Nmap

Sin la naturaleza propia de la herramienta tiene sus fallas (es una herramienta vieja después de todo) y es posible sortear estas contramedidas haciendo uso de escaneos TCP NULL, FIN y XMAS. ``portspoof`` espera recibir un paquete ``SYN`` en los puertos "camuflados" al inicio de una conexión TCP para luego aparentar que el puerto esta abierto y enviar información falsa en el *banner*, pero no espera recibir paquetes malformados o que se inicie una conexión TCP con paquetes diferentes al usual ``SYN``. En caso de recibir un paquete diferente al usual ``SYN``, la herramienta no tendrá efecto y el *host* se vera forzado a enviar una respuesta conforme al RFC correspondiente.

### Escaneos TCP NULL, FIN y XMAS

Los escaneos TCP NULL, FIN y XMAS deben su nombre a que envían paquetes sin banderas o con banderas especiales, siempre excluyendo la bandera ``SYN``, que es usada para iniciar un *three-way handshake*. De esta forma, estos escaneos son útiles para evadir firewalls configurados para bloquear paquetes TCP con la bandera ``SYN`` y por tanto conexiones TCP entrantes, al igual son considerados mas sigilosos que un escaneo ``SYN`` ya que evitan el posible registro de conexiones TCP al nunca enviar un paquete ``SYN`` e iniciar un *three-way handshake*.

Para nuestro caso, usaremos estos escaneos para sortear las contramedidas de ``portspoof`` y probar los puertos abiertos del servidor web haciendo uso de Nmap de forma rápida y eficaz. Estos tres escaneos esperan recibir un paquete TCP ``RST`` si el puerto se encuentra cerrado, o ningún paquete de vuelta si el puerto esta abierto. Cabe aclarar que un si no se envía de vuelta un paquete de respuesta, esto también es un indicativo de que el paquete enviado inicialmente fue filtrado por un firewall, por tanto Nmap identificara el puerto como "abierto o filtrado".

Por supuesto los escaneos TCP NULL, FIN y XMAS no son la panacea para evitar firewalls e IDS modernos. Y no son capaces de identificar correctamente si un puerto esta abierto o no en sistemas como Microsoft Windows y dispositivos Cisco, ya que estos y otros sistemas enviaran un paquete ``RST`` al recibir un paquete ``ACK``, independientemente de si el puerto esta abierto o no.

### Usando el escaneo TCP NULL para evadir portspoof

Con Nmap podemos hacer escaneos NULL, FIN y XMAS haciendo uso de las banderas ``-sN``, ``-sF`` y ``-sX``, respectivamente. Usaremos un escaneo NULL al ser el mas simple, aunque los tres servirán para nuestro propósito.

![nmap-null-scan](/blog/como-evadir-portspoof-con-nmap/nmap-null-scan.png)

> El puerto 4444 es el puerto donde opera el demonio de portspoof.

Finalmente logramos identificar los puertos que realmente están abiertos (o filtrados). Para verificar si realmente están filtrados podemos usar un escaneo TCP ACK con la bandera ``-sA``. Si el puerto se encuentra abierto o cerrado, responderá con un paquete ``RST``. De lo contrario, no enviara de vuelta una respuesta.

![nmap-ack-scan](/blog/como-evadir-portspoof-con-nmap/nmap-ack-scan.png)

Verificamos que los puertos no se encuentran filtrados, por lo que ahora podemos realizar escaneo de versiones y hacer uso del Nmap Scripting Engine con confianza.

![nmap-version-real-scan](/blog/como-evadir-portspoof-con-nmap/nmap-version-real-scan.png)

> No obtuvimos información valiosa con este escaneo, pero valía la pena realizarlo sabiendo que estos puertos realmente están abiertos.

Ahora que nuestra fase de escaneo ha concluido, podemos proceder a enumerar individualmente estos servicios y continuar con nuestra prueba de penetración.
