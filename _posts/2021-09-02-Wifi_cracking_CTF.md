---
title: Wifi cracking CTF (RedteamRD)
classes: wide
categories:
  - Post
tags:
  - JamielSP
  - CTF
  - Wifi
toc: true
last_modified_at: 2021-09-17T15:42:38-04:00
---
![logo](/assets/images/posts/2021-09-02-Wifi_cracking_CTFlog.jpg)

El CTF consistió en los siguientes tres retos principales:

```bash
1. Cuáles son los dispositivos conectados al 1er Access Point (AP-Guess - está abierto).
2. Conseguir credenciales del File CakeGuest_028ddb17732e.
3. Conseguir credenciales del file Swagger_6cb0ce434217.
```

**PCAPS utilizados por si quieren replicar ustedes localmente.**

[Cake pcap](/assets/flies/posts/2021-09-02-Wifi_cracking_CTF/CakeGuest_028ddb17732e.pcap)

[Swagger](/assets/flies/posts/2021-09-02-Wifi_cracking_CTF/Swagger_6cb0ce434217.pcap)

### **Overview**

En general me gustó mucho aprendí mucho sobre este mundillo de lo que se conoce hoy como redes inalámbricas, algunas terminologías que desconocía, y gracias al meet previo al CTF pude conocer e interesarme mucho más sobre este tema. En mi mente pensaba que las redes inalámbricas hoy en día estaban muy seguras, pero me di cuenta que no :)

### **Terminología**

**PMKID** es el identificador único de la clave utilizado por el AP para hacer un seguimiento de la PMK que se está utilizando para el cliente. PMKID es un derivado de AP MAC, Client MAC, PMK y PMK Name.

**BSSID** (Basic Service Set  Identifier), de una red de área local inalámbrica (WLAN), es un nombre  de identificación único de todos los paquetes de una red inalámbrica  para identificarlos como parte de esa red.

**SSID** es una secuencia de un máximo de 32 octetos incluida en todos los [paquetes](https://es.wikipedia.org/wiki/Paquete_de_red) de una [red inalámbrica](https://es.wikipedia.org/wiki/Red_inalámbrica) para identificarlos como parte de esa red. El código consiste en un máximo de 32 [caracteres](https://es.wikipedia.org/wiki/Carácter_(tipo_de_dato)), que la mayoría de las veces son [alfanuméricos](https://es.wikipedia.org/wiki/Alfanumérico) (aunque el estándar no lo especifica, así que puede consistir en  cualquier carácter). Todos los dispositivos inalámbricos que intentan  comunicarse entre sí deben compartir el mismo SSID.

**Beacon** es uno de los marcos de administración en **redes** inalámbricas **WLAN** basadas en IEEE 802.11. Los Beacon frames contienen toda la información sobre la **red** inalámbrica y son transmitidos periódicamente para anunciar la presencia de la **red WLAN**.

**Airodump-ng**  se utiliza para la captura de paquetes de tramas 802.11 sin procesar y es especialmente adecuado para recoger los IVs (vector de inicialización) de WEP con la intención de utilizarlos con aircrack-ng. Si tienes un receptor GPS conectado al ordenador, airodump-ng es capaz de registrar las coordenadas de los puntos de acceso encontrados. 

**Aircrack-ng** es una suite de software de seguridad inalámbrica. Consiste en un  analizador de paquetes de redes, recupera contraseñas WEP y WPA/WPA2-PSK y otro conjunto de herramientas de auditoría inalámbrica.

**Estos son solo uno de los pocos terminos que se escucharon en el meet de todas formas aquí les dejo el Meetup, por si tienen el placer de pasarse por allá, no hay desperdicio :)**

**Ahora si, hora del hacking !!!!!!**

![loco](/assets/images/posts/2021-09-02-Wifi_cracking_CTFloco.gif)

## Reto 1

### **Cuáles son los dispositivos conectados al 1er Access Point (AP-Guess - está abierto)**

Esto es relativamente fácil de realizar, primero haremos uso de la herramienta **airdump-ng** para detectar los APS que tenemos en la zona, lo realizaremos por medio de la wlan0 que es la interfaz que configuramos en modo monitor.

```bash
airodump-ng wlan0
```

![reto1_1](/assets/images/posts/2021-09-02-Wifi_cracking_CTFreto1_1.png)

Visto y detectado el objetivo de nosotros que es el **AP-GUESS**, podremos ver su **BSSID**, con el cual vamos a realizar el próximo ataque que nos dará lo que queremos. El BSSID de nuestro AP objetivo fue el siguiente, **02:ED:1F:44:EF:42**, ahora con el siguiente comando, vamos a detectar todos los dispositivos que se encuentran conectados a esa red, solo tenemos que ejecutar lo siguiente.

```bash
airodump-ng --bssid 02:ED:1F:44:EF:42 --channel 6 --write cap wlan0
```

Con este comando obtenemos el siguiente resultado:

![reto1_2](/assets/images/posts/2021-09-02-Wifi_cracking_CTFreto1_2.png)

Obteniendo las MACS de todos los dispositivos de esa red, el siguiente paso es identificar los dispositivos, en mi caso las marcas de estos, yo lo obtuve por medio de la siguiente página. [Mac Address an OUI Lookup](https://aruljohn.com/mac.pl)

74:9E:AF:0C:33:F2: Apple, Inc.

50:7A:C5:0C:33:F2: Apple, Inc

44:07:0B:0C:33:F2:  Google, Inc

C4:93:D9:47:A2:80: Samsung Electronics Co.,Ltd

7C:23:02:82:CE:EB: Samsung Electronics Co.,Ltd

F8:95:EA:02:25:16: Apple, In

**Con esto ya hemos completado el reto número 1 :)**

![compelte](/assets/images/posts/2021-09-02-Wifi_cracking_CTFcomplete.gif)

## Reto 2

### **Conseguir credenciales del File CakeGuest_028ddb17732e**

-------------------------

Algo obligatorio en este CTF era hacer uso de **hashcat** para poder desifrar la password de del WIFI, por lo tanto si tenemos un poco de exp, sabemos que hashcat no trabaja con archivos PCAPS como los que nos proporcionaron, así que debemos buscar algún método para convertir este **PCAP**, en un formato complatible con hashcat. 

#### **Cambiando formato**

Con un poco de búsqueda en internet encontramos una forma, y es que descargándonos el repositorio de **hashcat-utils**, podemos hacer uso de un **binario** que nos permite hacer la conversión que queremos. Nos clonamos el repositorio con el siguiente comando.

```bash
git clone https://github.com/hashcat/hashcat-utils
```

Descargado el repositorio, accederemos a un dir llamado **src**, estado ahí podremos observar un binario llamado **cap2hccapx**, es el que nos ayudará a cumplir nuestro cometido. Solo tendremos que insertar el siguiente comando estando en el dir donde se encuentra el **.bin**.

```bash
./cap2hccpax.bin [file.pcap] [convertido.hccapx]
```

![reto2](/assets/images/posts/2021-09-02-Wifi_cracking_CTFreto2.png)

Veremos que obtuvimos el archivo convertido e incluso nos dice que el file contiene 2 WPA Handshake, muy bien !!!

Ahora nos viene la pregunta ¿Qué sigue?, crackear el file obtenido..., correcto !!!

Insertando el siguiente comando haciendo uso de hashcat obtendremos nuestro resultado deseado.

```bash
hashcat -m 2500 [file.hccapx] /usr/share/wordlist/rockyou.txt
```

En el comando vemos que estamos haciendo uso del módulo 2500 de hashcat este módulo su nombre es **WPA-EAPOL-PBKDF2**, específicamente para crackear este tipo de protocolos de red, luego le pasamos el file que convertimos y por último le pasamos el wordlist que vamos a usar para crackear. Después de esperar un buen rato (esto va a depender de los recursos de tu PC), obtenemos el resultado.

![reto2_2](/assets/images/posts/2021-09-02-Wifi_cracking_CTFreto2_2.png)

**Lo logramos :)**

![complete2](/assets/images/posts/2021-09-02-Wifi_cracking_CTFcomplete2.gif)

## Reto 3

### **Conseguir credenciales del file Swagger_6cb0ce434217**

---

Vemos que si intentamos convertir el PCAP de este reto a formato **hccapx** observaremos que el file no contiene ningún handshake, ya desde ese punto vas notando que no se puede hacer el método anterior para conseguir la password. 

Por lo que vamos a intentar extraer el PMKID, para así de esta manera obtener la password, esto lo haremos haciendo uso de la tool llamada **hcxpcaptool**, esta nos permite extraer PMKID de PCAPS, y guardarlo en un file que debe de tener la extensión **.16800**. Por lo tanto el comando se vería de la siguiente manera.

```bash 
hcxpcaptool -z nombre_del_file.16800 {ruta del pcap}
```

![reto3](/assets/images/posts/2021-09-02-Wifi_cracking_CTFreto3.png)

![reto31](/assets/images/posts/2021-09-02-Wifi_cracking_CTFreto3_1.png)

Vemos que obtuvimos el **PMKID** de manera efectiva, ahora solo nos queda ponernos hack y obtener la password.

Ahora con hashcat en este caso utilizaremos el módulo 16800, que corresponde al módulo de PMKID, por lo tanto nuestro comando estaría contruido de la siguiente manera.

```bash
hashcat -m 16800  [file.16800] [wordlist]
```

![reto3_2](/assets/images/posts/2021-09-02-Wifi_cracking_CTFreto3_2.png)

Lo hemos hecho nuevamente, somos unos cracks !!!!

![complete3](/assets/images/posts/2021-09-02-Wifi_cracking_CTFcomplete3.gif)

