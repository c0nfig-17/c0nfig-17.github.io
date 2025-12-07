---
title: Funcionamiento y evasión de MotW
description: Mark of the Down es un control sobre la procedencia de ficheros, entraremos en el artículo a como funciona y como evadirlo.
date: 2025-12-07
categories: [evasion]
tags: [bypass, email, evasion, local, ]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/motw/0.png

---
## 1. ¿Que es Mark of the Web?

En muchas ocasiones descargamos ficheros de internet y "mágicamente" nuestro ordenador detecta que ha sido descargado de internet y nos impide ejecutarlo ¿porque?

![Desktop View](/assets/img/motw/1.png)

En Microsoft elaboraron una manera de marcar la procedencia de los ficheros de cara a poder evitar comportamientos maliciosos partiendo de la procedencia del fichero. Para ello lo que se desarrolló es la insercción de un metadato llamado Zone Identifier como parte del [**Alternate Data Stream(ADS)**](https://en.wikipedia.org/wiki/NTFS#Alternate_data_stream_(ADS)) el cual categoriza el archivo de su procedencia en base a 5 posibles categorias: </br>

0 = My Computer </br>
1 = Local intranet</br>
2 = Trusted sites </br>
3 = Internet </br>
4 = Restricted sites </br>

Podeís ver una descripción de cada una de las zonas en este [**enlace de Microsoft**](https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/platform-apis/ms537183(v=vs.85)) aunque son clasificaciones bastantes autodescriptivas. </br>

Puedes ver en tu propio ordenador en el panel de control el apartado que define luego realmente MotW y añadir sitios confiables. 
![Desktop View](/assets/img/motw/2.png)


## 2. ¿y mi fichero que?

Bueno vamos a comprobar cual es el estado de un fichero que descarmos de internet, en mi caso un instalador de Notepad++ mediante powershell. </br>

```text
PS C:\Users\c0nfig\Downloads> Get-Content -Path .\npp.8.8.8.Installer.x64.exe -Stream Zone.Identifier
[ZoneTransfer]
ZoneId=3
ReferrerUrl=https://notepad-plus-plus.org/
HostUrl=https://release-assets.githubusercontent.com/github-production-release-asset/33014811/71731d1b-bd3e-4ad8-9835-cdda378f5599?sp=r&sv=2018-11-09&sr=b&spr=https&se=2025-12-07T00%3A03%3A02Z&rscd=attachment%3B+filename%3Dnpp.8.8.8.Installer.x64.exe&rsct=application%2Foctet-stream&skoid=96c2d410-5711-43a1-aedd-ab1947aa7ab0&sktid=398a6654-997b-47e9-b12b-9515b896b4de&skt=2025-12-06T23%3A03%3A01Z&ske=2025-12-07T00%3A03%3A02Z&sks=b&skv=2018-11-09&sig=PiGdEv%2BAgRZgJi6b8q0qQpWy2a0iXqj50n8GoHfAbmQ%3D&jwt=eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJnaXRodWIuY29tIiwiYXVkIjoicmVsZWFzZS1hc3NldHMuZ2l0aHVidXNlcmNvbnRlbnQuY29tIiwia2V5Ijoia2V5MSIsImV4cCI6MTc2NTA2MzgwOCwibmJmIjoxNzY1MDYzNTA4LCJwYXRoIjoicmVsZWFzZWFzc2V0cHJvZHVjdGlvbi5ibG9iLmNvcmUud2luZG93cy5uZXQifQ.gsXGWxyC_FTEBCZzBwewTpcqoNbLWzQK-KiQXK1Fgm4&response-content-disposition=attachment%3B%20filename%3Dnpp.8.8.8.Installer.x64.exe&response-content-type=application%2Foctet-stream
```

Como puedes ver el identificador de mi instalable es la Zona 3 que es Internet por lo que de primeras las herramientas de seguridad que tenga desplegadas desconfiaran del fichero. Tienes que tener en cuenta que además existen extensiones que son especialmente consideradas de alto riesgo y que es importante que sepas cuando planificas la descarga de un fichero. 

```text
.ade, .adp, .app, .asp, .cer, .chm, .cnt, .crt, .csh, .der, .fxp, .gadget, .grp, .hlp, .hpj, .img, .inf, .ins, .iso, .isp, .its, .js, .jse, .ksh, .mad, .maf, .mag, .mam, .maq, .mar, .mas, .mat, .mau, .mav, .maw, .mcf, .mda, .mdb, .mde, .mdt, .mdw, .mdz, .msc, .msh, .msh1, .msh1xml, .msh2, .msh2xml, .mshxml, .msp, .mst, .msu, .ops, .pcd, .pl, .plg, .prf, .prg, .printerexport, .ps1, .ps1xml, .ps2, .ps2xml, .psc1, .psc2, .psd1, .psm1, .pst, .scf, .sct, .shb, .shs, .theme, .tmp, .url, .vbe, .vbp, .vbs, .vhd, .vhdx, .vsmacros, .vsw, .webpnp, .ws, .wsc, .wsf, .wsh, .xnk
```


## 3. Desbloqueo de ficheros
En este punto tendremos que ver como desbloquear el fichero, podemos hacerlo de manera sencilla yendo a las propiedades del documento y pulsando en Desbloquear.
![Desktop View](/assets/img/motw/3.png)

Una vez lo haces ese fichero se desbloquea a los controles que ayudarían a identificar el archivo. 
Existe un comando mediante powershell para desbloquear el fichero y [**Microsoft**](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/unblock-file?view=powershell-7.5) lo documenta. Es bastante sencillo de usarlo:

```text
Unblock-File -Path .\npp.8.8.8.Installer.x64.exe
```

También puedes "simplemente" eliminarlo

```text
Remove-Item -Path .\npp.8.8.8.Installer.x64.exe
```

## 4. Bypasses y estrategias de evasión

### 4.1 Propagación de MotW y software no compatible
Un punto que es necesario entender es que MotW aveces no es soportado o sobre todo no propagado. Por ejemplo 7z hasta hace relativamente poco no marcaba MotW y era perfecto para distribuir payloads y se usaba mucho en Malware. Existe un recurso que os puede ayudar para ver maneras de introducir payloads de manera comparada que es el [**repositorio archiver-MOTW-support-comparison**](https://github.com/nmantani/archiver-MOTW-support-comparison)

### 4.2 Tipo de fichero y packing
Puedes hacer packing de tu fichero de manera sencilla para evitar la detección via MotW. Es muy conocido el recurso [**PackMyPayload**](https://github.com/mgeeky/PackMyPayload) que te ayuda con python a automatizarlo. Ten en cuenta que usar ficheros .rar o .zip con contraseña suele levantar indicadores en algunos entornos al ocultar el contenido, puede ser importante en algunos puntos dar algo más de visibilidad entendiendo que alertas de comportamiento vas a despertar.

### 4.3 Uso de .LNK y .HTA para evasión
En muchas ocasiones puedes usar .LNK por ejemplo en vez del .exe dado que los formatos son diferentes y podemos evitar los controles de Smart Screen. Te paso un analisis de [**Unit 42 sobre malware usando lnk**](https://unit42.paloaltonetworks.com/lnk-malware/) que vislumbra un poco como usarlo. 

### 4.4 Evasiones adicionales
Voy a recomendar dos evasiones explicadas por mr.d0x en su blog en los post [**FileFix - A ClickFix Alternative**](https://mrd0x.com/filefix-clickfix-alternative/) y [**FileFix (Part 2)**](https://mrd0x.com/filefix-part-2/) que proporciona evasiones concretas a MotW que pueden ser interesantes. 

![Desktop View](/assets/img/motw/4.jpg)

## 5. Consideraciones generales como OPSEC
Tienes que tener en cuenta que lo principal en el momento en el que estás tratando de desplegar información descargada es evitar tener que generar nada que no sea necesario. ¿Porque te vas a descargar un fichero de internet si no es necesario en la fase en la que estás? Tienes que profundizar en la necesidad que tienes a la hora de reliazar este paso y como lo vas a realizar. En este post nos hemos centrado mucho en entender como funciona y como saltarlo pero no hemos entrado en 1)¿Es necesario en el punto del ejercicio que estoy realizando? y 2)¿Desde donde voy a descargar toda la información que necesito?. Si estás en medio de un ejercicio y no tienes estos puntos en mente puedes comenter fallos tontos e imprudencias que desde luego el atacante no realizaría. Si quieres profundizar un poco más en esto te dejo por aquí otro artículo sobre el tema [**OPSEC Tricks en delivery**](https://c0nfig17.com/posts/OPSEC-Tricks-en-delivery/) por si te es de ayuda.


---
## Apoya el contenido de ciberseguridad en castellano

Si esta publicación te ha sido útil y quieres apoyar mi trabajo para que continúe creando más contenido, aquí te dejo algunas formas de apoyar:

1. **Compartir el contenido**  📲
   Si crees que esta guía puede ser útil para otras personas, compartirla en tus redes sociales es una gran ayuda. 

2. **Donar en Ko-fi**  💖
   Puedes hacer una donación rápida a través de Ko-fi para ayudarme a seguir publicando guías y tutoriales. ¡Cada aportación cuenta y es muy apreciada! 

   <script type='text/javascript' src='https://storage.ko-fi.com/cdn/widget/Widget_2.js'></script><script type='text/javascript'>kofiwidget2.init('Apoya este contenido!', '#455d85', 'A0A41BO608');kofiwidget2.draw();</script> 
   
3. **Usa mi enlace de afiliado de NordVPN y NordPass para mejorar tu seguridad y apoyar la creación de contenido**  🛡️
   Puedes suscribirte a [**NordVPN**](https://go.nordvpn.net/aff_c?offer_id=15&aff_id=132246&url_id=
