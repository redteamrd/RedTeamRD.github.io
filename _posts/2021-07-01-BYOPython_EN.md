---
title: My process to discover how to use Dropbox pythonNN.dll to execute Python scripts without Python.
classes: wide
---
## Introduction
I was playing Warzone and start noticing some packet loss, which is "no bueno" if you are playing an FPS game. I went into the task manager and look for those processes that were using the network and start killing everything, like Rambo. On Dropbox.exe I clicked "Open File Location" and once the folder opened, I notice some Python related files. 

![image](https://user-images.githubusercontent.com/29603107/124169512-50b47f80-da74-11eb-8e82-467e94be1e59.png)

After I finish playing, I went back to the folder and notice a file that caught my attention: **python38.dll**

![image](https://user-images.githubusercontent.com/29603107/124169764-9bce9280-da74-11eb-9171-04c10e089b8a.png)

Quickly I went into Twitter and search for "python.dll [@byt3bl33d3r](https://github.com/byt3bl33d3r)", whenever I think on Python for offensive usage, his name (Marcello) is on top of my head. You can read more about him on his [Twitter](https://twitter.com/byt3bl33d3r) or [GitHub](https://github.com/byt3bl33d3r). The search shows only one result:

![image](https://user-images.githubusercontent.com/29603107/124169956-d6382f80-da74-11eb-83ad-8561cdf03b23.png)
[Twitter image link](https://twitter.com/byt3bl33d3r/status/1381128038474084353?s=20)

So, I was like, mmm this should work! I may be able to use python38.dll to execute a Python script without Python on the system. 
 
I start searching "how to execute Python with python.dll" and some links from docs.python.org appeared in my search results:  
  •	[Python on Windows FAQ — Python 3.9.5 documentation](https://docs.python.org/3/faq/windows.html#how-can-i-embed-python-into-a-windows-application)  
  •	[Embedding Python in Another Application — Python 3.9.5 documentation](https://docs.python.org/3/extending/embedding.html)  

The [documentation](https://docs.python.org/3/faq/windows.html) said: "Run-time linking greatly simplifies link options; everything happens at run time. Your code must load pythonNN.dll using the Windows **LoadLibraryEx()** routine. The code must also use access routines and data in pythonNN.dll (that is, Python’s C API’s) using pointers obtained by the Windows **GetProcAddress()** routine. Macros can make using these pointers transparent to any C code that calls routines in Python’s C API"
 
Ok, I can load Dropbox pythonNN.dll with **LoadLibrary()** on my own process and use **GetProcAddress()** to access its methods, interesting. Now how can I execute code using python? This is where the 2nd link come into play [Embedding Python in Another Application](https://docs.python.org/3/extending/embedding.html)
 
With the example provided [1.1. Very High Level Embedding](https://docs.python.org/3/extending/embedding.html#very-high-level-embedding) I know I can use **PyRun_SimpleString()** function to run Python commands like: 

`PyRun_SimpleString("print('Hello from python')").` 

## Building the program
With the information I got I was ready to build a C# program which:  
1.	**LoadLibrary()** – to import pythonNN.dll from Dropbox into my program.  
2.	**GetProcAddress()** – to retreive **PyRun_SimpleString()**, **Py_Initialize()** and **Py_Finalize()** functions address.  
3.	Call those function using **[DInvoke](https://github.com/TheWover/DInvoke)**.  

Let’s explain how everything works:

1 – **[LoadLibrary()](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya)** loads the specified module (in our case Dropbox pythonNN.dll), into the address space of the calling process. 

To use this API within C# we need to (1) import the DLL that has the function (kernel32.dll) and (2) call the API function. You can use [pinvoke.net website]( https://www.pinvoke.net/default.aspx/kernel32/LoadLibrary.html) to see examples on how to call API from C#.

```
// (1) - Importing kernel32.dll and declaring the API call LoadLibrary
[DllImport("kernel32.dll")]
static extern IntPtr LoadLibrary(string name);

// (2) – Call the API function
var pyDll = LoadLibrary(PathToPythondll);
```

2 – **[GetProcAddress()](https://docs.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-getprocaddress)** - Retrieves the address of an exported function or variable from the specified dynamic-link library (DLL). 

To use this API within C# we need to (1) import the DLL that has the function (kernel32.dll) and (2) call the API function saving the function address location in a variable.

```
// (1) - Importing kernel32.dll and declaring the API call LoadLibrary
[DllImport("kernel32.dll")]
static extern IntPtr GetProcAddress(IntPtr hModule, string procName);

// (2) – Call the API function
var pyrunAddr = GetProcAddress(pyDll, "PyRun_SimpleString");
var pyInitAddr = GetProcAddress(pyDll, "Py_Initialize");
var pyFinAddr = GetProcAddress(pyDll, "Py_FinalizeEx");
```

What those functions are doing? 

**[Py_Initialize()](https://docs.python.org/3/c-api/init.html#c.Py_Initialize)** - In an application embedding Python, the **Py_Initialize()** function must be called before using any other Python/C API functions; with the exception of a few functions and the global configuration variables.

**[PyRun_SimpleString()](https://docs.python.org/3/c-api/veryhigh.html#c.PyRun_SimpleString)** – Executes the Python source code from command in the __main__ module according to the flags argument. If __main__ does not already exist, it is created. Returns 0 on success or -1 if an exception was raised. If there was an error, there is no way to get the exception information.

**[Py_FinalizeEx()](https://docs.python.org/3/c-api/init.html#c.Py_FinalizeEx)** - Undo all initializations made by **Py_Initialize()** and subsequent use of Python/C API functions, and destroy all sub-interpreters that were created and not yet destroyed since the last call to **Py_Initialize()**. Ideally, this frees all memory allocated by the Python interpreter. Normally the return value is 0. If there were errors during finalization (flushing buffered data), -1 is returned.

Basically we need to call **Py_Initialize()** function before using **PyRun_SimpleString()** and to end our Python script "correctly" we should use **Py_FinalizeEx()**.

3 - **[DInvoke](https://github.com/TheWover/DInvoke)** – Now that we have the address location of those functions, we need a way to invoke those functions. With **DInvoke** we can dynamically invoke unmanaged API without PInvoke (the method we used for **LoadLibrary()** and **GetProcAddress()**). The use case for DInvoke is that the Python DLL location may change based on the Dropbox version.

To use DInvoke we need to (1) define the function we want to use, (2) get the memory address location of those functions (we did it with GetProcAddress()), and (3) initialize/invoke those functions. 

```
// (1) - Define the functions
[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
private delegate int PyRun_SimpleString(string command);
 
[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
private delegate void Py_Initialize();
 
[UnmanagedFunctionPointer(CallingConvention.Cdecl)]
private delegate void Py_Finalize();

// (2) – Get the memory address of the functions
var pyrunAddr = GetProcAddress(pyDll, "PyRun_SimpleString");
var pyInitAddr = GetProcAddress(pyDll, "Py_Initialize");
var pyFinAddr = GetProcAddress(pyDll, "Py_FinalizeEx");

// (3) Initialize & Invoke Functions
Py_Initialize Py_Initialize = (Py_Initialize)Marshal.GetDelegateForFunctionPointer(pyInitAddr, typeof(Py_Initialize));
Py_Initialize(); 
 
PyRun_SimpleString PyRun_SimpleString = (PyRun_SimpleString)Marshal.GetDelegateForFunctionPointer(pyrunAddr, typeof(PyRun_SimpleString));
string pythonCode = @"print('Hello from python')";
int result = PyRun_SimpleString(pythonCode);
 
Py_Finalize Py_Finalize = (Py_Finalize)Marshal.GetDelegateForFunctionPointer(pyFinAddr, typeof(Py_Finalize));
Py_Finalize();
```

The full code looks like this:

```
using System;
using System.Runtime.InteropServices;
 
namespace BYOPython
{
    class Program
    {
	  
        [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
        private delegate int PyRun_SimpleString(string command);
 
        [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
        private delegate void Py_Initialize();
 
        [UnmanagedFunctionPointer(CallingConvention.Cdecl)]
        private delegate void Py_FinalizeEx();
 
        static void Main(string[] args)
        {
            var pythonPath = @"C:\Program Files (x86)\Dropbox\Client\123.4.4832";
 
            string pythondll = $@"{pythonPath}\python38.dll";
 
            Console.WriteLine($"[+] Loading pythonNN.dll from Dropbox directory");
            var pyDll = LoadLibrary(pythondll);
 
            Console.WriteLine($"[+] Getting Functions Addresses.");
            var pyrunAddr = GetProcAddress(pyDll, "PyRun_SimpleString");
            var pyInitAddr = GetProcAddress(pyDll, "Py_Initialize");
            var pyFinAddr = GetProcAddress(pyDll, "Py_FinalizeEx");
 
            Py_Initialize Py_Initialize = (Py_Initialize)Marshal.GetDelegateForFunctionPointer(pyInitAddr, typeof(Py_Initialize));
 
            Console.WriteLine($"[+] Initializing Python.");
            Py_Initialize();
 
            Console.WriteLine($"[+] Setting Python Payload.");
            PyRun_SimpleString PyRun_SimpleString = (PyRun_SimpleString)Marshal.GetDelegateForFunctionPointer(pyrunAddr, typeof(PyRun_SimpleString));
 
            string pythonCode = @"print('Hello from python')";
 
            Console.WriteLine($"[+] Executing Python Payload.");
            int result = PyRun_SimpleString(pythonCode);
 
            Py_FinalizeEx Py_FinalizeEx = (Py_FinalizeEx)Marshal.GetDelegateForFunctionPointer(pyFinAddr, typeof(Py_FinalizeEx));
 
            Console.WriteLine($"[+] Finalizing Python.");
            Py_FinalizeEx();
        }
 
        [DllImport("kernel32.dll")]
        static extern IntPtr LoadLibrary(string name);
 
        [DllImport("kernel32.dll")]
        static extern IntPtr GetProcAddress(IntPtr hModule, string procName);
    }
}

```

I compiled the code using x86, same as Dropbox python38.dll, and execute it, but when it calls **Py_Initialize()** it crashes, with the error: **"ModuleNotFoundError: No module named 'encoding'**. 

![image](https://user-images.githubusercontent.com/29603107/124183945-e9ec9180-da86-11eb-8d09-8bf1f357c39c.png)

Ok ok, when I'm using Python, I typically solve this issue with **pip install ModuleName**, but how I'm supposed to do this here?
 
I went into the Python website and download the original [Python Windows embeddable package](https://www.python.org/ftp/python/3.9.6/python-3.9.6-embed-amd64.zip) to see if my code works and if the issue it's related to the Dropbox DLL. After unzipping [Python Windows embeddable package](https://www.python.org/ftp/python/3.9.6/python-3.9.6-embed-amd64.zip) on C:\Python38, I changed the DLL PATH to C:\Python38\python38.dll and ran my code:

![image](https://user-images.githubusercontent.com/29603107/124184103-1a343000-da87-11eb-9dab-209640fe649c.png)

Yeah!! It worked, but why it doesn't work with Dropbox python38.dll? I opened [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) and set a filter for my app: CallingPython.exe 

![image](https://user-images.githubusercontent.com/29603107/124184301-5ebfcb80-da87-11eb-8ef9-ca50745a8514.png)

While using the original python38.dll, [Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon) show it accesses C:\Python38\python38.zip, looks like it assumes there's a .zip file with the same name as the DLL.

![image](https://user-images.githubusercontent.com/29603107/124184327-6a12f700-da87-11eb-88f8-6faa54b577fd.png)

I was wondering, what's the purpose of this .zip file? While looking around I found this information on [Python documentation website](https://www.python.org/dev/peps/pep-0273/): "Currently, sys.path is a list of directory names as strings. If this PEP is implemented, an item of sys.path can be a string naming a zip file archive. The zip archive can contain a subdirectory structure to support package imports. The zip archive satisfies imports exactly as a subdirectory would" 

So, for our purpose Python uses this zip to load modules, including the module "encodings", the one we got an error before. In the following image you can partially see the content of python38.zip.

![image](https://user-images.githubusercontent.com/29603107/124184430-8c0c7980-da87-11eb-85b1-e8c1bcf16e34.png)

Python38.zip doesn’t exist on Dropbox’s directory, but there’s a **python-packages.zip** which is the same, with a different name, as you can see in the following image.  

![image](https://user-images.githubusercontent.com/29603107/124184476-9d558600-da87-11eb-9c45-eb04f570bc1d.png)

To use that package with our Dropbox python38.dll, I set the program's environment variables the same as the python38.dll Path and include python-packages.zip as PATH too. 

```
Environment.SetEnviromentVariable("PYTHONHOME", pythonPath);
Environment.SetEnviromentVariable("PYTHONPATH", $@"{pythonPath}\python-packages.zip");
```

After crossing my fingers and pray to God, it worked!!! But while yelling Alleluia!! I got another error **ModuleNotFoundError: No module named "site"**!! Oh my God!!!!! ☹

![image](https://user-images.githubusercontent.com/29603107/124184550-beb67200-da87-11eb-8015-82a4cb1ccd64.png)

This is where StackOverflow and [CristiFati](https://stackoverflow.com/users/4788546/cristifati) come to the rescue. CristiFati indicates that the **Py_Initialize()** function checks if **Py_NoSiteFlag** is 0 and then loads the module **site**, so to avoid it he mentions you can set **Py_NoSiteFlag** to 1 and the site module won’t be loaded. You can learn more about it [here](https://stackoverflow.com/questions/39539089/what-files-are-required-for-py-initialize-to-run).

To change **Py_NoSiteFlag** value in memory what I did was use **GetProcAddress()** to retrieve the memory location and **Marshal.Copy** to set it to 1. 

```
var pyNoSiteFlagAddr = GetProcAddress(pyDll, "Py_NoSiteFlag");

int[] variable = new int[1];
Marshal.Copy(pyNoSiteFlagAddr, variable, 0, 1); // copying Py_NoSiteFlag value to the variable.

variable[0] = 1; // 0 for False, 1 for True
Marshal.Copy(variable, 0, pyNoSiteFlagAddr, 1); // setting Py_NoSiteFlag to 1
```

I added this piece of code, compile it, and wowowo!! I was able to execute Python using Dropbox installation files 😊 

![image](https://user-images.githubusercontent.com/29603107/124184768-0dfca280-da88-11eb-9a39-6e9cc214019a.png)

I know, I know, you want to see a shell pop up and. There you have!

{% include video id="lZAZBkmIpY4" provider="youtube" %}

## Bring your own Python. 

After playing a little bit with this code, I ended up realizing that I can use any program that does the same as Dropbox, you can go into your **Program Files/Program Files (x86)** directories and search for "python*.dll". I found another software on my computer that also brings its python38.dll 😊

![image](https://user-images.githubusercontent.com/29603107/124184872-2f5d8e80-da88-11eb-8d9b-548ac2617a9e.png)

But what if I don’t have any application that brings its pythonNN.dll? Well you can **B**ring **Y**our **O**wn **P**ython and execute your scripts 😊 

In the following example, I’m downloading Python 3.8 from python.org and executing my Python code 😊 

{% include video id="rHkBq-ru-yw" provider="youtube" %}

On my [GitHub](https://github.com/juliourena) you will find the source code to abuse Dropbox pythonNN.dll and a reference on how to **B**ring **Y**our **O**wn **P**ython, keep in mind you may need to change the path of your Dropbox installation to make it work.

[BYOPython Github repository](https://github.com/juliourena/BYOPython)

If you want a live video me explaining how I discover it and reproducing it, take a look at [HackTheBox - RedTeamRD Meetup - Spectra & Abuso de Dropbox python.dll](https://www.youtube.com/watch?v=oXb3hRZrKrA&t=4265s) (Spanish).

**Note**: I reported this to Dropbox but it was out of scope: "Attacks requiring physical access to a user's device."

Special thanks to tekwizz123 for helping me out reviewing the blog post.

**God bless you!**

**Serving Christ is not a task, but a relationship. God's Friends Jn 15: 15**
