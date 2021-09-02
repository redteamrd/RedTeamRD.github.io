---
title: Buenas Practicas para Firewalls / NGFW
---
Al momento de realizar un Pentest es mucho más fácil aprovecharse del desconocimiento de los administradores de redes/seguridad que ignoran cómo podemos utilizar diferentes técnicas para comprometer las redes, aunque haya Firewalls configurados.
 
Esta información les será útil a los Pentesters y a los encargados de defender la red.
 
Muchos piensan que con la instalación de Firewalls o NGFW (Next Generation Firewalls), estarán protegidos al 100% de los ataques de red, no obstante, aunque los NGFW han puesto una capa extra de complejidad a los APT (Advance Persistent Threat), no cabe duda de que si estos no están bien configurados es posible hacerles un bypass. 
 
Cuando hablamos de Hacking y Bypass de Firewalls difícilmente nos referiremos a abrir un puerto que esté cerrado en el Firewall y así conseguir acceso local, regularmente hablaremos de formas de utilizar los Firewalls y sus configuraciones para pasar tráfico malicioso a la organización.
 
Para entender cómo hacerle Bypass al Firewall hagamos un poco de teoría y discutamos sobre **Bind TCP** y **Reverse TCP**, estos son dos de las formas que utilizamos para establecer la comunicación entre el equipo afectado y la máquina del atacante, no obstante, no son las únicas formas de hacerlo. 
 
Al utilizar **Bind TCP** el atacante se conecta al equipo de la víctima. El malware está diseñado para abrir un puerto en la máquina afectada, y este puerto es utilizado por el atacante para comunicarse con ese equipo. Los Firewalls/NGFW bloquearán estos ataques, porque el puerto es abierto en el equipo afectado no en el Firewall.
 
![Bind_TCP](/assets/images/bind_tcp.png)
 
En el caso del **Reverse TCP** el equipo afectado se comunica con el equipo del atacante. En este caso el Malware hace que el equipo afectado se comunique constantemente con el equipo del atacante.
 
![Reverse_TCP](/assets/images/reverse_tcp.png) 
 
Por esta razón los atacantes preferirán utilizar el **Reverse TCP** ya que no necesitan abrir puertos en el Firewall, se pueden comunicar con un equipo externo utilizando puertos conocidos como HTTP (TCP 80) o HTTPS (TCP 443) que comúnmente están permitidos en las organizaciones. 
 
Aquí es donde hacen sentido los NGFW por encima de los Firewalls que no son capaces de hacer inspección de tráfico por si solos, un NGFW tendrá la capacidad de evaluar el contenido del tráfico. Si usas un Firewall tradicional es muy probable que cualquier malware pueda pasar por tu red sin ningún problema. La alternativa para esto es utilizar IPS/IDS para monitorear el tráfico de red. 
 
# Recomendación #1 - Activar IPS/IDS en todo el tráfico entrante y saliente. 
 
Sin las funcionalidades de IPS/IDS lamentablemente estamos expuestos a cualquier tipo de Malware, es por esto por lo que es importante habilitar IPS/IDS no solo en el tráfico que va a desde el internet a los servidores, sino también en el de los clientes al Internet. Por el tema del Reverse TCP
 
![Analisis_Network](/assets/images/ips_ids_ssl_inspection.png)
 
# Recomendación #2 - Bloquear todo, permitir solo lo necesario.
 
Este tiende a ser una de las medidas de seguridad más comunes, pero no se sorprendan que muchas organizaciones tienden a simplificarse la vida utilizando el **Allow ALL** o en ocasiones sucede que, al momento de un problema de comunicación, para resolver rápido y **"arreglar después"** utilizan el **Allow ALL** y el después, lamentablemente, nunca llega. Esta brecha podría ser utilizada por los atacantes para comprometer la organización. Veamos un ejemplo de porque esto es un problema:
 
Tenemos el protocolo **SMB** o **TCP 445**, es un protocolo utilizado comúnmente para comunicaciones internas y no recomendado para conexiones públicas, ya que se pueden utilizar muchos para robo de credenciales o bypass de antivirus. 
 
Supongamos que el usuario X recibe un pdf por correo.

![phishing](/assets/images/email_phishing.png)
 
Al usuario abrir el pdf este enviará el hash en forma de **NTLMv2**. Este hash puede ser crackeado y obtener el password del usuario, de modo que este puerto es recomendable que esté bloqueado.

![NTLMv2-hash](/assets/images/NTLMv2-hash.png)

Para la creación del PDF utilicé [bad-pdf](https://github.com/deepzec/Bad-Pdf), para recibir el hash la aplicación [Responder](https://github.com/SpiderLabs/Responder) y el password puede ser crackeado utilizando John The Ripper (Instalado en Kali por defecto). 
 
# Recomendación #3 - Bloquear sitios desconocidos. 
 
Algunas soluciones de NGFW permiten hacer filtrado web, y siempre encontraremos la categoría **Desconocido**, **No registrado** o cualquier otra cosa que haga referencia a que un dominio en particular que aún no ha sido categorizado por la poca información del mismo. ¿Por qué bloquear este tráfico? 
 
Regularmente nos encontraremos que un atacante querrá hacer ingeniería social a la empresa, si el atacante no cuenta con un sitio web categorizado, tendrá que comprar un sitio nuevo sin categorización para crear su campaña de phishing, si nos mantenemos bloqueando los sitios desconocidos podríamos evitar estas campañas de phishing. 
 
El concepto detrás de esto es algo parecido al proyecto de [WhatsApp-Protector](https://github.com/juliourena/WhatsApp-Protector) que realicé, para más información seguir el enlace.
 
# Recomendación #4 - Habilitar SSL Inspection en todos los puertos permitidos.
 
Algunos NGFW por defecto solo hacen inspección SSL a comunicaciones que utilizan SSL/SSH, dígase HTTPS, SMTPS, IMAPS, FTPS, etc.; sin embargo, esto podría permitir a un atacante utilizar el protocolo HTTP o puerto TCP 80 para pasar tráfico cifrado sin que esté siendo validado por la inspección SSL. 
 
Un atacante podría utilizar el link https://myhackersite.com:80 y este tráfico no sería inspeccionado por consecuencia un atacante podría hacer Bypass a cualquier IPS/IDS. Si permites otros puertos que no sean SSL aun así deberías inspeccionar esos puertos.
 
**Nota**: Los IPS/IDS comúnmente utilizan firmas digitales para identificar patrones en el tráfico, es posible que, aunque tengan estos habilitados algún tipo de Malware desconocido pueda pasar por la red sin ser detectado por el IPS/IDS, es por esta razón que deben ser considerados otros controles como protección a los equipos, para incrementar la seguridad. **Un NGFW no lo es todo**.
 
Espero que les sea útil, cualquier duda o sugerencia no duden en escribir.

**Dios les bendiga!**

**El servir a Cristo, no es una tarea, sino una relación. Amigos de Dios. Jn 15:15** 
