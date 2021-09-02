---
title: “WhatsApp Protector”
---
Automatizar es la razón por la que me encanta la programación, aunque debo confesar que no soy un buen Developer, no puedo negar que me gusta mucho. 
 
Una amiga queriendo ayudar aquellos que no tienen trabajo, comparte un enlace de una oportunidad de trabajo en el grupo de la iglesia, sin saber que el mismo era un engaño, el famoso phishing, la ingeniería social que tantos problemas nos da. Inmediatamente abro el enlace y confirmo que se trata de algo malicioso y advierto a los demás que no entren. En ese momento, algunos opinaron y pensé, que tal si puedo crear una aplicación que se conecte a mi WhatsApp y valide la categoría a la que pertenece el enlace. Entiendo que esto no vencerá el phishing, aquellos hackers que toman páginas con certificados expirados o compran sitios que tienen categorías regularmente permitidas, pero creo que servirá para reducir el impacto que tienen algunas campañas de phishing conocidas.
 
En un futuro post planeo hablar un poco de como identificar phishing en las páginas web, correos o mensajes, en caso de que no quieran verificar la categoría o en el caso que el sitio esté categorizado inapropiadamente.
 
Entendiendo que este programa no busca "Eliminar el Phishing de WhatsApp" lo considero una prueba de concepto, que permitirá a otros a crear herramientas útiles para automatizar ciertas cosas en WhatsApp, como hacer una tarea que cada 5 minutos te mande un Email a todas tus cuentas si tienes un mensaje de tu esposa que no has contestado 😊

A continuación, presentaré la forma en que desarrollé el programa y como ustedes pueden utilizarlo. Para la verificación de las categorías utilicé el servicio público de Fortinet (marca que conozco y utilizo) pueden visitar [FortiGuard WebFilter](https://www.fortiguard.com/webfilter) para probarlo. Agradecimientos a Fortinet por mantener este servicio público. Es importante señalar que, aunque utilizo la web de Fortinet, este desarrollo no está ligado a la compañía, pudieran utilizar cualquier recurso público para esa verificación. 

# Lenguaje de Programación. 

Al principio tenía la intención de hacerlo en C#, incluso complete la parte de revisar las categorías al pasar un enlace. Lo pensé hacer utilizando el FortiGate 80CM que tengo en casa, pero decidí implementarlo web, por si alguien quisiera probar.

![csharp](/assets/images/c-sharp-categorychecker.png)

Aquí el código:
```console
using System;
using System.Collections.Generic;
using System.ComponentModel;
using System.Data;
using System.Drawing;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using System.Windows.Forms;
using System.Net;
using System.IO;
using System.Text.RegularExpressions;
 
namespace Whatsapp_Fortinet
{
    public partial class Form1 : Form
    {
        public Form1()
        {
            InitializeComponent();
        }
 
        private void btnCheckWebsite_Click(object sender, EventArgs e)
        {
            HttpWebRequest request = (HttpWebRequest)WebRequest.Create("https://fortiguard.com/webfilter?q=" + txtWebsite.Text);
            request.Method = "GET";
 
            HttpWebResponse response = (HttpWebResponse)request.GetResponse();
            using (StreamReader reader = new StreamReader(response.GetResponseStream()))
            {
                string result = reader.ReadToEnd();
 
                var match = Regex.Match(result, "Category: (.*) />");
                lblCategory.Text = match.Groups[1].Value.Replace("\"","");
                
            }
        }
    }
}
```

En mi búsqueda no encontré una librería de C# que me permitiera conectarme a WhatsApp, lo que si encontré fue una en Python, no quería mezclar ambos lenguajes o hacer algún tipo de integración, por eso no pensé en completar el código en C#. Para mi suerte, Python y C# son los lenguajes que hablo. Así que sin más pensarlo decidí implementarlo en Python.
 
La librería que encontré fue esta: [WebWhatsapp-Wrapper](https://github.com/mukulhase/WebWhatsapp-Wrapper) es un proyecto de [Mukul Hase](https://github.com/mukulhase) les invito a ver su trabajo.
 
Leí el [README](https://github.com/mukulhase/WebWhatsapp-Wrapper/blob/master/README.md), pero algunas cosas no me quedaron muy claras y soy de las personas que le gusta usar **print('variable')** para ver si funciona o no, jeje, así que tenía que probar. 
 
# Instalación de Librerías y demás.
 
Utilicé Ubuntu x64 para esto, pero podrían usar cualquier distribución, los ejemplos y comandos están pensados para la última actualización de Ubuntu. Procedí a instalar python3 y webwhatsapi, si no los tienen pueden instalarlo utilizando los siguientes comandos:

```console
sudo apt install python3
sudo apt install python3-pip
pip3 install webwhatsapi
 ```

Necesitamos también descargar el [Gecko Driver](https://github.com/mozilla/geckodriver) que permite interactuar con los navegadores, con la ayuda de este driver es que podremos trabajar con web.whatsapp.com y ubicarlo en algún directorio del $PATH. Para descargar [aquí](https://github.com/mozilla/geckodriver/releases)
 
```console
wget https://github.com/mozilla/geckodriver/releases/download/v0.20.1/geckodriver-v0.20.1-linux64.tar.gz
tar -xvzf geckodriver-v0.20.1-linux64.tar.gz
chmod +x geckodriver
sudo mv ./geckodriver /usr/local/bin/.
 ```

Ya con esto podemos empezar nuestro código. El código está dividido en dos partes, la 1ra hace la conexión a Web WhatsApp y la 2da identifica los enlaces y su categoría.
 
# Conexión de WhatsApp
 
Necesitamos importar las librerías de WhatsAPIDriver, entramos a la consola de python3 y ponemos lo siguiente:
 
```console
from webwhatsapi import WhatsAPIDriver
from webwhatsapi.objects.message import Message
 ```

Ahora procedemos a crear una instancia de la clase WhatsAPIDriver que nos permitirá interactuar con nuestro WhatsApp web:

```console
driver = WhatsAPIDriver(client='firefox',loadstyles=True,profile='/home/plaintext/dev/profile-whatsapp')
```

**Notas Importantes:**
* Pueden utilizar otros navegadores como Chrome, pero Firefox es el default. 
* Loadstyles les permitirá ver el contenido de la página con sus estilos, tarde mucho en encontrar esto porque la página de web.whatsapp.com sin esto se mostraba incompleta. 
* Profile nos permitirá guardar la sesión de web.whatsapp.com de modo que no tengamos que leer el QR cada vez que lancemos la aplicación.
 
Ahora podemos ver nuestro estado:
 
`driver.is_logged_in()`
 
Buscar los chats que tenemos por número de teléfono:
 
`driver.get_chat_from_phone_number('18491112222')`
 
Solo les dejaré eso para que lo tengan como referencia, pueden seguir investigando que otras informaciones sacar de su WhatsApp en el [proyecto de WebWhatsapp-Wrapper](https://github.com/mukulhase/WebWhatsapp-Wrapper).
 
# Verificación de Categorías en URL
 
Si visitan la página de FortiGuard podrán ver la opción de consulta de WebFiltering [FortiGuard WebFiltering](https://fortiguard.com/webfilter) si probamos por ejemplo: [http://plaintext.do](http://plaintext.do) nos indica que nuestra página está categorizada como Information Technology :)
 
![fortiguard](/assets/images/fortigard-webfilter.jpg)
 
Ahora lo que necesitamos replicar esta misma consulta en Python e imprimir la categoría. Utilizaremos la librería *requests*, una forma sencilla sería:
 
```console
import requests, re
url = 'http://plaintext.do'
r = requests.get('https://fortiguard.com/webfilter?q=' + url)
category = re.findall("Category: (.*) />",r.text)[0].replace("\"","")
print(category)
```

Aquí utilizo la librería *re*, que nos permite hacer expresiones regulares para extraer del texto lo que necesitamos. En este caso, estoy buscando la palabra *Category:* 
 
Otra cosa importante para la que utilizaremos expresiones regulares, es para extraer todas las URLs de los mensajes de WhatsApp, para ello utilizaremos un Regex creado por [rcompton](https://github.com/rcompton), pueden ver su proyecto [urlmarker.py](https://github.com/rcompton/ryancompton.net/blob/master/assets/praw_drugs/urlmarker.py)
 
Ahora que tenemos la categoría, ¡solo resta mezclar todo!
 
# WhatsApp Protector 
 
WhatsApp Protector, creo que me excedí con el nombre, pero los proyectos de desarrollo son como los hijos, le pones el nombre que más te guste y este me gusto. 
 
![whatsapp-protector](/assets/images/whatsapp-protector.png)
 
**Opciones:**

**Chat_Id** - Necesitamos el Chat_Id para poder elegir la conversación que queremos monitorear por enlaces, solo configuré uno, pero pueden modificar el código para agregar más.
 
**Buscar** - Si no saben el Chat_Id, pueden utilizar la opción -b o --buscar para indicar el nombre de la conversación (puede ser una persona o grupo).

**Tiempo** - Nos permite definir cuantos segundos durará la ejecución del programa. 
 
**Directorio** - Se utiliza para grabar la sesión, de tal forma que no tengan que estar escaneando el QR siempre. El único inconveniente es que cada vez que inicia, abre un nuevo navegador. Antes de utilizar la opción son necesarios algunos pasos. Basados en la guía de [codemanat](https://github.com/codemanat/WebWhatsAPI/blob/master/README.md)
 
# Configuración de sesión permanente: 

**1ro - Abrir la consola de python3 y poner lo siguiente:**
 
```console
from webwhatsapi import WhatsAPIDriver
from webwhatsapi.objects.message import Message
driver = WhatsAPIDriver(client='Firefox',loadstyles=True)
```

**2do - Escanear el código QR y cerrar la consola utilizando CTRL + C.**
 
**3ro - Crear un perfil de Firefox.**

* En la consola (bash) poner lo siguiente: `firefox -p`
* Crear Perfil
* Seleccionar un nombre
* Elegir el directorio (Crear un directorio donde se vayan a guardar los registros de Firefox)
* Entrar a [https://web.whatsapp.com](https://web.whatsapp.com) y escanear el código QR
* Entrar nuevamente a [https://web.whatsapp.com](https://web.whatsapp.com) y validar si la sesión persiste.
* Finalizar.
 
Ahora solo resta utilizar el programa:
 
# Búsqueda de Chat_id

Para buscar el chat_id utilizamos:

```terminal
python3 whatsapp-protector.py -d /home/plaintext/dev/plaintext-profile -b vitilla
```

![whatsapp-protector-busqueda](/assets/images/ws-busqueda.png)

Y ya tenemos el chat_id de dos conversaciones que tienen el nombre **vitilla**

# Protección de Chat

Para ejecutar la protección solo sería, recuerden que la -t es opcional 😊

```terminal
python3 whatsapp-protector.py -d /home/plaintext/dev/plaintext-profile -c 18000070508-1500082004@g.us -t 120
```

![whatsapp-protector-busqueda](/assets/images/ws-protector.png)

Y así se vería en el Whatsapp 😊

![whatsapp-protector-action](/assets/images/whatsapp-action.png)

Pueden encontrar el proyecto aquí:

[WhatsApp-Protector](https://github.com/juliourena/WhatsApp-Protector)

Espero que les sea útil, cualquier duda o sugerencia no duden en escribir.

**Dios les bendiga!**

**El servir a Cristo, no es una tarea, sino una relación. Amigos de Dios. Jn 15:15** 
