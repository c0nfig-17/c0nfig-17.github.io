---
title: ES Password Craking e Inteligencia para Red Teams
date: 2025-06-02
categories: [Passwords]
tags: [passwords, hashes]     # Los tags deben estar siempre en minúsculas.

---


## 1. ¿De verdad mi contraseña no puede ser Admin1234?
En el día a día trabajando como Ethical Hacker uno ve y escucha cosas que generan cierto miedo... y también que nuestro trabajo (y el de los atacantes) sea más sencillo. Las contraseñas débiles suelen estar a la orden del día, y desde luego los usuarios no suelen estar de acuerdo con reforzar este aspecto (o asumen las consecuencias sin conocerlas). <br>
Durante ejercicios de Red Teaming y Pentesting es normal que haya buenas prácticas y una preparación avanzada de algunos temas, pero en ocasiones existe cierto desprecio por el cracking offline de hashes o se considera que existen ténicas menos costosas que usar. Si bien esto no es falso, es necesario que tengamos preparado un diccionario robusto y consistente para que todo lo que pase por nuestras manos sea cuanto antes vulnerado, ya que normalmente vamos con el tiempo acotado (problema que no tiene un atacante). <br>
Trato de enseñaros algunas de las cosas aprendidas para mejorar los diccionarios que tenemos que elaborar y hacerlos más funcionales, extensos y realistas para con cualquier target, sobre todo cuando salimos de un CTF y deja de valer el Rockyou y pa' dentro.


## 2. Diferencias entre la fase de inteligencia y ser inteligente
En muchas ocasiones durante ejercicios de Red Team sobre todo realizamos se realiza una fase de Inteligencia en muchos ejercicios, y los que esteís bajo [`DORA`](https://www.hackplayers.com/2025/01/dora-tlpt-ciberseguridad-financiera.html) , sabeís que esta fase es obligatoria para realizar el ejercicio. Si bien se trabaja mucho la superficie de ataque y los elemenos que ayudarán a, si conseguis entrar, dar pasos hacia vulnerar la compañía creo que no se suele trabajar tanto y de manera inteligente algunos factores con los que vas contrarreloj segúnn entres a la compañía. <br>
¿Cuanto tiempo tardarás en crackear ese lssas dumpeado? ¿Rockyou y rezar? ¿Confiamos todo a la RT5090? <br>
Desde luego no parece lo mejor, y hay que preparar no solo los recursos básicos. Aquí no entraremos a hablar de Rockyou de [`SecLists`](https://github.com/danielmiessler/SecLists/tree/master) porque ya deberías conocerlo. Rockyou es un complemento a muchas otras, de SecLists y otros repos que deberias usar como [`Weakpass`](https://weakpass.com/) o muchos más. Cada maestrillo tiene su librillo.  <br>
Dicho todo esto ¿que puedo añadir a dentro de la fase de inteligencia como indicadores para la elaboración de un diccionario completo?


| Elementos                    | Notas            |
| :--------------------------- | :--------------- |
| Ciudades                     | Puedes incluir todas las ciudades de un país, pero ¿todos los pueblos de la comunidad donde opera esta compañía?|
| Deportes                     | ¿Son de Fútbol, Rugby o les va el basket? Toda información sobre equipos y jugadores puede ser relevante|
| Música                       | ¿Hay un estilo particular o cantantes en la zona en la que se opera? Quizás te sorpendes con pass:Rosalia1234|
| ¿En que trabajan?            | ¿Son una empresa de pasteles? ¿o son una empresa de moda? Como la contraseña sea CamisetaSinMangas vamos a reirnos  |
| ¿En que piensas mientras trabajas? | ¿Te pasas el día mirando por la Ventana1234 o tienes el MoNitoRRR muy cerca? ¿Probablemnete se pasan el día pensando en las Vaca-Ci0NES?  |
| Mi jefe y el compañero       | Hay gente que reutiliza nombres de usuarios con los que trabaj (mención aparte nombres de hijos, padres, hermanos...) puede ser algo interesante a introducir|
| Estructura de usuario        | Los usuarios de la empresa son ID00058 ¿Que pasa si mi usuario es id123400058= |
| ¿Usan alguna tecnología concreta? | Alomejor la contraseña del compi de diseño es AdobeAdobe_Adobe2025 ¿probable no? |
| ¿Que prooveedores trabajan allí? | Cuanto te juegas a que el externo de Microsoft tiene como contraseña algo de Microsoft... |

En general hay que echarle un poco de cara. Aumentar el diccionario pensando en ¿que podría de manera real poner un trabajador de contraseña? es la clave para conseguir que la fase de craking sea más funcional. Desde luego la experiencia nos demuestra que mejorar esta fase nos ayuda a ir más rápido cuando los tiempos aprietan.

## 3. Scrapping web y perfilado
Suele ser muy interesante obtener de cada una de las webs, o IPs públicas, de una compañía extraer las palabras más interesantes. Yo no sé sobre ropa, si estoy atacando a Zara necesitaré información sobre que palabras pueden ser usadas en su negocio ¿que mejor que su web? probablemente haya palabras muy usadas, tanto por proveedores, ciudades, prendas, máquinas... <br>
Vamos al lio. Al principio nos aseguraremos de tener disponible en nuestra máquina CeWL, el cual podemos obtener de Github mediante el siguiente [`repo`](https://github.com/digininja/CeWL) y consultar su uso desde el propio [`manual`](https://www.kali.org/tools/cewl/) de kali . 
En caso de no tenerlo disponible usaremos el siguiente comando para instalarlo en nuestro equipo:

```kali
sudo apt install cewl
```
Nosotros lo que haremos será tirar el siguiente comando, con el cual nos aseguramos que se recogen palabras de maneraas de 6 digitos las cuales se incluyen los emails que localice y me los guarda en un fichero. Además podriamos utilizar el apartado de metadatos que nos puede ser útil para servicios de infra. Usamos la menor potencia del spider por el momento para no generar ningún tipo de impacto.

> No es común pero se consciente que puedes hacer algo de ruido lanzando muchas peticiones. Valora dependiendo del tipo de auditoria los saltos que des. 
{: .prompt-warning }

```kali
cewl -m 6 -d 1 -e -v -w /home/usuario/mi_fichero https://www.c0nfigwebsuperchulaymuycara.com/
```

En ese momento ya tendré "mi_fichero" con el que habré obtenido gran cantidad de palabras relaccionadas con la entidad

> Recuerda que debes realizar esto con cada una de las compañías o páginas web del mismo grupo empresarial. www.c0nfigwebsuperchulaymuycara.com es interesante pero también c0nfig.com o empleo.c0nfig.com
{: .prompt-tip }


## 4. Password Policy y optimización
Muchas veces es fundamental saber que política de contraseñas existe realmente en un entorno para no perder tiempo de cracking en la elaboración de la lógica.
> Es muy importante que sepas que no todos los sistemas de la entidad que auditas puede que estén correctament bastionados. Quizás la contraseña "Admin" no cumple los requisitos en el dominio pero si que en una aplicación del tercero o en una máquina concreta. No descartes a la ligera y reevalua dependiendo del contexto.  
{: .prompt-warning }

Existen algunas comprobaciones que son fundamentales, ya que ¿si consigues crackearlo y no tienes tiempo? Com CMD y Powershell podemos evaluarlo
 ```cmd
C:\C0nfig>net accounts
Tiempo antes del cierre forzado:                  2
Duración mín. de contraseña (días):               2
Duración máx. de contraseña (días):               365
Longitud mínima de contraseña:                    8
Duración del historial de contraseñas:            Ninguna
Umbral de bloqueo:                                5
Duración de bloqueo (minutos):                    5
Ventana de obs. de bloqueo (minutos):             5
Rol del servidor:                                 WORKSTATION
```

Con esto sabemos que previsiblemente una contraseña inferior a 8 caracteres no debería ser válida. Existen muchos otras formas de sacar esto y en cada entorno de una manera particular. Puedes por ejemplo simplemente tratar de cambiar la contraseña de un usuario random e ir evaluando que falla y que no, ya que tiende a validar la contraseña nueva antes de saber si cumples el criterio de la anterior. 

Puedes usar herramientas como [`PowerView`](https://github.com/PowerShellMafia/PowerSploit) también para enumerar las políticas del dominio con powershell. 

```kali
powershell Get-DomainPolicyData | select -expand SystemAccess
```

## 5. kwprocessor y la regla del mínimo esfuerzo
Una manera de contribuir al diccionario que vamos a elaborar es utilizar herramientas como [`kwprocessor`](https://github.com/hashcat/kwprocessor) que te permiten generar configuraciones de keyboardwalks dependiendo del idioma del teclado del target. 

![Desktop View](/assets/img/password_cracking/pass_1.png) <br>

## 6. Iteraciones en el diccionario
Una vez conseguimos un gran volumen de información podemos tratar de mejorarla realizando iteraciones sobre el diccionario que hemos hecho combinando diferentes elemenyos y patrones que están normalizados. Para esto podemos usar CUPP [`Cupp`](https://github.com/Mebus/cupp.git) <br>
Simplemente lo inciamos y comenzamos a trabajar.
```kali
python3 cupp.py -w /home/andi/Desktop/mi_fichero
```
La herramienta es clara y muy sencilla de usar
```kali
    > Do you want to concatenate all words from wordlist? Y/[N]: n
    > Do you want to add special chars at the end of words? Y/[N]: n
    > Do you want to add some random numbers at the end of words? Y/[N]:
    > Leet mode? (i.e. leet = 1337) Y/[N]
```
Tras seleccionar todas las opciones se nos generará un fichero llamado `mi_fichero.cupp.txt` el cual tendrá nuestro nuevo diccionario.
Yo para trabajar trato de ser bastante metódico para no dejarme nada. En este caso concreto me he definido 4 niveles de diccionario que puedo configurar en base a las transformaciones que haga y el volumen. Así elijo un poco que usar. Cuando paso del Easy al Big por ejemplo lo que hago es elimitar las iteraciones que ya he comprobado en el Easy para no volver a perder 2 horas (o lo que tardes) en algo que sabes que no existe.

| **Niveles** | **Transformaciones** | **Ejemplos** |
| ---- | ---- | ---- |
| **Easy** | - Concatenar palabras | C0nfigHacking |
| **Medium** | - Concatenar palabras<br>- Añadir caracteres especiales al final de las palabras | C0nfigHacking_* |
| **Big** | - Concatenar palabras<br>- Añadir caracteres especiales al final de las palabras<br>- Añadir números random al final de las palabras | C0nfigHacking2025_* |
| **Insane** | - Concatenar palabras<br>- Añadir caracteres especiales al final de las palabras<br>- Añadir números random al final de las palabras<br>- Leet mode | C0nfigH4ck¡ng2025_* |

> Ten en cuenta que dependiendo de la infrastructura que tengas podrás levantar más hilos y será más sencillo o menos usar este volumen de contraseñas. No siempre más es mejor, si tienes una gráfica de hace varios años puede que no se lo mejor y tengas que optimizar todo. Pero si tienes un rack de RTX5090 trabajando en paralelo con buena ventilación y mantenimiento... Aprovechalo! No te quedes con la duda de que hay detrás de ese hash! 
{: .prompt-info }

## 7. Hashcat y uso avanzado
Durante el propio uso de hashcat puedes utilizar funciones las cuales no son del todo habituales para mejorar el crackeo de las contraseñas usando diferentes funcionalidades. 
Puedes usar reglas por ejemplo para añadir años a las contraseñas.

```kali
hashcat -a 0 -m 1000 hash.txt mi_fichero.cupp.txt -r rules\add-year.rule
```
Una manera de ser más eficiente es utilizar mascaras para por ejemplo usar mayusculas y minúsculas en las mismas posiciones siempre

```kali
hashcat -a 3 -m 1000 C:\usr\hash.txt ?u?l?l?l?l?l?l?l?d
```
Puedes combinar diccionarios usando hashcat

```kali
hashcat -a 1 -m 1000 hash.txt mi_fichero.cupp.txt diccionario_posterior.txt -j $- -k $!
```
Puedes usar combinaciones de números aleatorios del 0000 al 9999.

```kali
hashcat -a 6 -m 1000 hash.txt mi_fichero.cupp.txt ?d?d?d?d
```
Si quieres que los números del del 0000 al 9999 vayan delante puedes usar.
```kali
hashcat -a 7 -m 1000 hash.txt mi_fichero.cupp.txt ?d?d?d?d
```

## 8. BonusTrack: Rainbow Tables y optimización
Una opción que cada vez va a ser más usada por nuestro lado de cara a mejorar los tiempos son las RainbowTables. De manera muy sencilla son hashes ya tratados de contraseñas. Cuando usamos Hashcat lo que hacemos es convertir una contraseña en un hash y posteriomente compararla con el hash que hemos sustraido. Pero ¿Que pasa si ya tenemos el hash y no saltamos la conversión?. Las RainbowTables nacen en base al cómputo previo de estas contraseñas, las cuales se transforman en hashes de NTLM por ejemplo y se comparan directamente. Es un proceso mucho más rápido en el momento del crackeo, lo cual supone no solo consumir menos tiempo sino también menos recursos como muestra la tabla posterior.<br>

![Desktop View](/assets/img/password_cracking/rainbow_2.png) <br>

Tiene algunas pegas como es que todo el proceso previo hay que realizarlo previamente, por lo que puede no ser óptimo para algo personalizado. Por otro lado su principal pega es el espacio que usa. Mientras que un diccionario customizado grande puede pesarte unos GBs, los Hashes alomejor pesan unos TBs. Por lo que hay que saber cuando usarlo. El proyecto de [`rainbowcrackalack`](https://www.rainbowcrackalack.com/) además de porporcionar información proporciona hashes que puedes usar así como algunos test que pueden ser de ayuda. 


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

