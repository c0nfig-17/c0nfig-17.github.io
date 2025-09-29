---
title: Esteganografía y hacking
description: ¿Que es la esteganografía? ¿Como podemos usarlo en ciberseguridad?
date: 2025-09-29
categories: [evasion]
tags: [evasion, esteganografía, ofuscación, criptografía]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/estego/stego4.png
---

## 1. ¿Que es Esteganografía?
La idea de este post es poder profundizar en la Esteganografía (en adelante "estego") y así poder analziar algunas funciones ofensivas así como poder entender desde el punto de vista defensivo su funcionamiento. Lo primero ¿que es esto de laa esteganografía? La definición más resumida nos la da [**Kaspersky**](https://latam.kaspersky.com/resource-center/definitions/what-is-steganography?srsltid=AfmBOoqmvmrnOcsevn2LeCWEAD81hj2u8rZ82sEs49tZB8wgNKtal5EN&utm_source=affiliate&utm_medium=cpa&utm_campaign=es-LA_Affiliate_acq_ona_afm__all_b2c_awin_affiliatelink________&aw_affid=1858990&awc=22032_1758546067_63b2748c929d7ce57b887445c70eb834) "La esteganografía es la práctica de ocultar información dentro de otro mensaje u objeto físico para evitar su detección". Dentro de esto se abren multitud de posibilidades para ocultar información dentro de otra y ocultarla a simple vista, muchas de ellas son en imágenes, video, audio...


## 2. Confusiones típicas: ofuscación y criptografía.
Vale, muy bien la definición pero.... ¿eso no era ofuscar? ¿no se parece a la criptografía?... Que puedan servirte para cosas similares no significa que sean lo mismo. Vamos a diferenciar un poco todo esto para acertar con los onbjetivos del post:
- Criptografía: [**AWS**](https://aws.amazon.com/es/what-is/cryptography/) en un post tiene una definición que parece útil "La criptografía es una práctica que consiste en proteger información mediante el uso de algoritmos codificados, hashes y firmas. La información puede estar en reposo, en tránsito o en uso". Vamos con un ejemplo:
La contraseña "Galleta$12345" ha sido cifrada mediante un algoritmo criptográfico el cual convierte la contraseña en el siguiente hash.

```text
usuario::DOMINIO:61dde887aeb2af2a:76DD8039B9606119586BC9A4EF5F3C1:010100000000000080C0653150DE09D20107929B9D6080F5BB000000000200080053004D004200330001001E00570049004E002D0056005600400140053004D00420033002E006C006F00630061006C0003001E00570049004E002D005600560053004D00420033002E006C006F0063006100070008008
```

- Ofuscación: Es un método o métodos para dificultar la comprensión de algo. Hace que sea dificil de obtener la información. Me parece que un buen ejemplo es el que muestran en su página de presentación en  [**Argfuscator**](https://argfuscator.net/about.html) por lo que os lo dejo por aquí
![Desktop View](/assets/img/estego/argfuscator.svg)

## 3. Profundizando en estego
Existen multitud de técnicas para realizar estego e incrustar información en otra como puede ser modificando los metadatos de archivos (de manera muy simple) o mediante LSB por ejemplo como se explica en el siguiente post [**Esteganografía LSB en imágenes y audio**](https://daniellerch.me/stego/intro/lsb-es/) . <br>
Para entender el concepto y poder profundizar un poco en él vamos a usar [**Steganography Tools**](https://futureboy.us/stegano/). La idea es sencilla vamos a utilizar la web para subiendo un logo y con una contraseña introuducir un payload
![Desktop View](/assets/img/estego/estego1.png)

Como vemos comparando el fichero que subimos en comparación con el que modificamos 
![Desktop View](/assets/img/estego/stego2.png)

La propia web nos permite comparar dos archivos que se supone son identicos y detectar las diferencias
![Desktop View](/assets/img/estego/stego3.png)

En este caso la página nos permite ver las diferencias de contenido en la imagen. 
![Desktop View](/assets/img/estego/stego4.png)

Con la propia web podemos ver la información que se ha ocultado en la web
![Desktop View](/assets/img/estego/stego5.png)
![Desktop View](/assets/img/estego/stego6.png)

En general creo que existen recursos sobre el tema (muchos de ellos te los dejo más abajo), pero quisiera descasar el trabajo de [**Daniel Lerch**](https://www.linkedin.com/in/daniellerch/) del cual teneís un blog que me ha ayudado en muchas ocasiones (y sobre todo cuando estuve estudiando el tema en profundidad) ya que baja mucho y explica con muchos ejemplos os dejo su [**blog aquí**](https://daniellerch.me/index-es/) ya que seguro que os es de ayuda

## 4. Algunas aplicaciones ofensivas
Lógicamente existen aplicaciones ofensivas además de para transmitir información sin más en la stego. Te doy un par de recursos. 

### 4.1 Shellcode injectors
Existen algunos recursos públicos como [**ImgPayload**](https://github.com/CyberSecurityUP/ImgPayload) basado en la idea de [**imgect: BMP/GIF Shellcode Injector**](https://github.com/0xZDH/imgect) que te permiten inyectar código en algunos ficheros. Viene bastante bien explicado como utilizarlos en cada uno de los repos por lo que os recomiendo meteros directamente a echarle un vistazo.

### 4.2 Loader de MaldevAcademy
Hace nada la gente de [**MaldevAcademy**](https://maldevacademy.com/) subió el repositorio [**MaldevAcademyLdr.2: RunPE implementation with multiple evasive techniques**](https://github.com/Maldev-Academy/MaldevAcademyLdr.2?tab=readme-ov-file) en el cual hay un módulo dedicado especificamente a ocultar payloads en ficheros PNG,JPG,JPEG,BMP y GIF.




**Te recomiendo algunos recursos para que puedas profundizar:**

**Esteganografía en imágenes digitales:** <br>
[**107 aplicaciones**](https://www.jjtc.com/Steganography/tools.html)
[**Digital invisible ink toolkit (BMP, JAVA, RS-attack, tutoriales)**](https://diit.sourceforge.net/)
[**Steghide**](https://steghide.sourceforge.net/)
[**OpenStego**](https://www.openstego.com/)
[**OpenPuff**](https://embeddedsw.net/OpenPuff_Steganography_Home.html)
[**Steg**](https://steg.drupalgardens.com/)
[**PQ**](https://dde.binghamton.edu/download/pq/)
[**YASS**](https://www.ws.binghamton.edu/fridrich/Research/yass_attack.pdf)
[**HxD, editor hexadecimal freeware**](https://mh-nexus.de/en/hxd/)
<br>

**Esteganografía en audio digital:** <br>
[**StegoWav**](https://packetstormsecurity.com/files/21655/stegowav.zip.html)
[**DeepSound**](https://jpinsoft.net/DeepSound/)
[**MP3Stego**](https://www.petitcolas.net/steganography/mp3stego/)
[**Enscribe**](https://www.coppercloudmusic.com/enscribe/)
[**Hide4PGP**](https://www.heinz-repp.onlinehome.de/Hide4PGP.htm)
<br>

**Esteganografía en vídeo digital:** <br>
[**MSU StegoVideo**](https://compression.ru/video/stego_video/index_en.html)
<br>

**Ocultación mediante técnica EoF:** <br>
[**Masker**](https://softpuls.weebly.com/)
[**StegoStick**](https://stegostick.sourceforge.net)
[**BDV DataHider**](https://www.bdvnotepad.com/products/bdv-datahider/)
[**Max File Encryption**](https://max-file-encryption.informer.com/)
[**OurSecret**](https://www.securekit.net/oursecret.html)
[**OmniHide Pro**](https://omnihide.com/)
<br>

**Esteganografía en SSOO, archivos y formatos de archivos:** <br>
[**S-Tools (módulo FDD)**](https://www.ljudmila.org/matej/privacy/kripto/stegodl.html)
[**bmap**](https://dl.packetstormsecurity.net/linux/security/bmap-1.0.17.tar.gz)
[**ComputerSecurityStudent - lesson1**](https://www.computersecuritystudent.com/FORENSICS/HIDING/lesson1/index.html)
[**StegFS**](https://github.com/albinoloverats/stegfs)
[**Truecrypt (hidden-volume doc)**](https://www.truecrypt71a.com/documentation/plausible-deniability/hidden-volume/)
[**VeraCrypt (Hidden Operating System)**](https://www.veracrypt.fr/en/VeraCrypt%20Hidden%20Operating%20System.html)
[**LADS**](https://www.aldeid.com/wiki/LADS)
[**Hydan**](https://www.autistici.org/crypto/index.php/remository/Steganography/linux/hydan-0.13-(linux)/)
[**steg86**](https://crates.io/crates/steg86)
[**wbStego4**](https://www.bailer.at/wbstego/pr_4ix0.htm)
[**InvisibleSecrets (pago)**](https://www.east-tec.com/invisiblesecrets/steganography-software/)
[**HTML-NINJA**](https://github.com/ephreet/html-ninja)
[**Deogol**](https://hord.ca/projects/deogol/intro.html)
<br>

**Esteganografía en redes:** <br>
[**StegTunnel**](https://sourceforge.net/projects/steg-tunnel/)
[**Covert_TCP**](https://dunnesec.wordpress.com/2014/12/10/covert_tcp-file-transfer-for-linux/)
[**ICMP (using ICMP to deliver shellcode)**](https://blog.romanrii.com/using-icmp-to-deliver-shellcode)
[**IPv6teal**](https://github.com/christophetd/IPv6teal)
[**dnssteganography**](https://github.com/adamcritchley/dnssteganography)
[**Mística**](https://github.com/IncideDigital/Mistica)
[**CloakifyFactory (Cloakify-Powershell)**](https://github.com/dumpsterfirevip/Cloakify-Powershell)
<br>

**Esteganografía lingüística:** <br>
[**ASCII**](https://pictureworthsthousandwords.appspot.com/)
[**Stelin**](https://stelin.sourceforge.net/)
[**stegUnicode**](https://github.com/mindcrypt/stegUnicode)
[**spam mimic**](https://www.spammimic.com/)
[**jano**](https://github.com/mindcrypt/jano)
[**Esteganografía lingüística y canales encubiertos - libro (PDF)**](https://github.com/mindcrypt/libros/blob/master/Esteganograf%C3%ADa%20ling%C3%BC%C3%ADstica%20y%20canales%20encubiertos%20-%20libro.pdf)
<br>

**Esteganografía en Internet y RRSS:** <br>
[**Powerglot**](https://github.com/mindcrypt/powerglot)
[**Hackaday - Shakespeare in a zip...**](https://hackaday.com/2018/11/07/shakespeare-in-a-zip-in-a-rar-hidden-in-an-image-on-twitter/)
[**Stegpage**](https://sourceforge.net/projects/stegpage/)
[**CameraShy**](https://sourceforge.net/projects/camerashy/)
[**covertchannels-steganography (GitHub)**](https://github.com/mindcrypt/covertchannels-steganography)
<br>



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
   
![Desktop View](/assets/img/Nordvpn/logonordvpn.png){: width="250"}

![Desktop View](/assets/img/Nordvpn/logonordpass.png){: width="200"}
---

¡Gracias por tu apoyo! 🙏
![Desktop View](/assets/img/banner.png) <br>
