---
title: Persistencia mediante Windows Terminal
description: Vamos a utilizar paso a paso la propia herramienta de Micrososft Windows Terminal para generar persistencia en un equipo comprometido.
date: 2025-11-16
categories: [Persistencia]
tags: [domain, persistencia, evasion, local ]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/terminal/terminal.png

---
## 1. Persistencia y OPSEC. No vayamos tan rápido.
Antes de entrar en materia (y para evitar meter la pata) vamos a hacer una pequeña explicación de que es esto de la "persistencia". Dentro de la killchain que seguimos como atacantes dentro de un ejercicio realizamos varias fases, en las cuales cumplimos objetivos diferentes antes de la ejecución de acciones (que es la última fase).

![Desktop View](/assets/img/terminal/killchain.png)

Dentro de esta cadena de acciones, cuando ya hemos conseguido realizar una explotación y alcanzado una máquina y elevado privilegios (o no), realizamos tecnicas de "persistencia". Las técnicas de persistencia que se encuentran en MITRE bajo [**TA0003**](https://attack.mitre.org/tactics/TA0003/) nos permiten mantener acceso a las máquinas comprometidas tanto si somos detectados como si tenemos cualquier problema con nuestra infrastructura. Normalmente estás técnicas suelen estar rodeadas de un gran estudio del [**OPSEC**](https://c0nfig17.com/posts/OPSEC-Tricks-en-delivery/) que necesitaremos para evitar ser detectados. Trataremos de no hacer ruido, hacer las llamadas justas, no desplegar lo que no necesitemos... en resumen tratar de hacer lo justo para poder mantenernos y sobre todo tener varias lineas de trabajo en caso de que nos detecten. Al fin y al cabo si hemos comprometido una máquina y desplegamos 3 métodos de persistencia y el analista solo ha conseguido detectar 2 seguiremos teniendo acceso a la máquina. Es por eso que las técnicas de persistencia suelen hacer enfasis en el OPSEC y en como se despliegan... a nadie le gusta estar trabajando en una brecha durante semanas o meses y que se vaya al traste porque has hecho una chapuza. <br>
En resumen: pase lo que pase queremos seguir teniendo acceso a esa máquina.

## 2. Terminal 2.0 usos y funcionalidades
En este post vamos a abusar de una herramienta tremendamente útil de Microsoft que viene ya por defecto en muchos ordenadores. [**Windows Terminal**](https://apps.microsoft.com/detail/9n0dx20hk701?hl=es-ES&gl=ES) es una solución sencilla de terminal con multiples funcionalidades que suele ser usada en entornos empresariales. Sin liarte mucho y desde la propia store de Microsoft la despliegas en un segundo y permites que todo el mundo trabaje de manera sencilla. Lo principal de esto es que puedes usar shells o lo que quieras realmente desde la aplicación, ya que si por ejemplo tienes WSL podrás usar un ubuntu o una kali sin problemas. A nosotros para este tutorial simplemente nos interesa como tal la interfaz de la terminal y entender el usuario prototipico de la misma: el desarrollador o el compi de IT. 

## 3. Persistencia mediante Windows Terminal
El método de persistencia que veremos lo he empezado a trabajar a partir del propio uso que le doy a la terminal y ver como puedo ejecutar código a partir de algunas de sus funciones, posteriormente me di cuenta de que existía más documentación sobre el mismo. Básicamente lo que vamos a realizar es inyectar una tarea secundaria en el inicio de una terminal de Windows Terminal al estilo de lo que realizaría [**SharPersist**](https://github.com/mandiant/SharPersist) con la técnica schtaskbackdoor. Lo que haremos será aprovecharnos de la facilidad de cambiar las rutas en la terminal, modificando las mismas para buscar ejecutar un beacon o la tarea que queramos. Esta ejecución nacerá del propio usuario, es decir dependerá de que el mismo inicie la sesión en la terminal, aunque podremos ver varias formas de realizarlo. <br> 
Os dejo un esquema de lo que realizaremos.

![Desktop View](/assets/img/terminal/1.png)

Como habeís visto, partiendo del contexto de un compromiso previo, la idea es envenenar un "perfil" de Windows Terminal. En mi caso para esta PoC lo que haré será ejecutar notepad. Lo primeor que haremos será abrir la terminal y nos iremos al apartado de configuración. Seleccionaremos el perfil que queremos envenenar (en el punto posyterior entraremos a detalles de configuración).

![Desktop View](/assets/img/terminal/2.png)

Lo que haremos será modificar la ruta que ejecuta y en vez de apuntar solo a la ejecución del cmd le diremos que abra notepad en esta PoC.

```text
%SystemRoot%\System32\cmd.exe /k "C:\Windows\notepad.exe"
```
El uso es francamente sencillo, simplemente abriremos la pestaña del perfil que hayas modificado y ya estaría 

![Desktop View](/assets/img/terminal/3.png)

> Parte de lo chulo de todo esto es que el proceso es independiente. Por lo que si el usuario finaliza la sesión en el tu has conseguido que el usuario ejecute el software y el servicio no se interrumpa.
{: .prompt-tip }

En este punto solo tienes una pega, que es que si el usuario ejecuta el simbolo de sistema en vez de la terminal no ejecutará el perfil concreto. Cuando el usuario ejecuta "simbolo de sistema" la ruta es la original y se abre en la terminal, pero si el usuario busca "terminal" o pulsa en el icono lo abrirá.
![Desktop View](/assets/img/terminal/4.png)

## 4. Mejoras en ejecución
Como vas viendo esto se puede ejecutar en multitud de causisticas, pero lo suyo es conocer los tips que nos permiten mejorar y estabilizar la ejecución de cara a mantener la persistencia. Vamos por partes

### 4.1 Perfil predeterminado
Uno de los puntos más importantes es conseguir la ejecución directa al abrir la terminal. La propia aplicación abre una cmd o powershell dependiendo de lo que haya escogido el usuario. En este punto puedes o envenenar todos los perfiles o envenenar solo el que se abre de manera predeterminada al ejecutar la sesión. 
![Desktop View](/assets/img/terminal/5.png)

> Ten en cuenta de que cada vez que el usuario abra una pestaña de la cmd ejecutará tu beacon. Envenenar todos los perfiles puede generar más ruido del necesario. Tenlo en cuenta por si quieres configurar un sleep en el C2 o una regla para la gestión de sesiones que te permita no generar un tráfico que llame demasiado la atención. 
{: .prompt-tip }

### 4.2 Iniciar al inicio del equipo
Si activas la siguiente opción la aplicación terminal se iniciará automáticamente al iniciar sesión el usuario, por lo que conseguirás una sesión de cada usuario que inicie sesión en la máquina ya que ejecutará el beacon en ese momento. Recuerda que debes tener envenenado el perfil predeterminado y tener la opción de iniciar en el perfil predeterminado, no en la última sesión.
![Desktop View](/assets/img/terminal/6.png)

### 4.3 Ejecución de perfil como administrador
Puedes combinar todo lo anterior con la capacidad de ejecutar perfiles de manera predeterminada como administrador o algunos en concreto. Si quieres que todo perfil abierto sea como administrador puedes hacerlo en valores predeterminados.

![Desktop View](/assets/img/terminal/7.png)

Si quieres hacerlo solo desde el perfil predeterminado puedes configurarlo para esto.

![Desktop View](/assets/img/terminal/8.png)

> Es importante entender que saltará el UAC, por lo que el usuario tendrá un paso "molesto" que realizar y que quizás ni siquiera ese usuario tiene permiso para activarlo. Además levantar muchas ejecuciones como administrador debido a que esté usando terminales todo el rato puede levantar alarmas.
{: .prompt-danger }

## 5. Facilidades para la modificación de perfiles

De cara a realizar automatizaciones y realizar todo esto en una shell, lo principal es entender donde se almacenan los perfiles de windows terminal y modificarlo. Para ello tienes un .json el cual consume directamente windows terminal con la información que necesitas en la siguiente ruta.

```text
$env:USERPROFILE\AppData\Local\Packages\Microsoft.WindowsTerminal_*\LocalState\settings.json
```
Dentro de settings.json encontrarás cada uno de los perfiles y podrás modificarlo. Para ello debes de buscar el campo "defaultProfile" dentro del JSON el cual indica el GUID del perfil que está por defecto. Posteriormente debes modificar el commandline del guid que quieres infectar con las acciones que quieras ejecutar.

```text
            {
                "commandline": "%SystemRoot%\\System32\\cmd.exe /k \"C:\\Windows\\notepad.exe\"\r",
                "guid": "{0cbb0axdd-73ve-6g78-a8ff-afcaaauu1645}",
                "hidden": false,
                "name": "S\u00edmbolo del sistema"
            },
```

En resumidas cuentas <br>
![Desktop View](/assets/img/terminal/9.png)

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
