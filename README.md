Full Guide: Autosplitter (Annelid) for Super Metroid on RetroArch Linux (Works Better Than Windows)

This step-by-step guide details how to configure Annelid (an autosplitter) to work with RetroArch (using the bsnes-mercury Performance core) on a Linux operating system (specifically Nobara 40, based on Fedora 40), enabling autosplitting for Super Metroid speedruns. The sni software bridge will be used for communication.

Objective: To enable Annelid to automatically detect in-game events in Super Metroid (via RetroArch) and automate your speedrun "splits".

Common Problem and Our Solution:
Are you a Super Metroid speedrunner on Linux and have you hit the frustrating wall of autosplitters? The lack of clear guides and compatibility issues (such as the challenges with compiling usbsnes or QUsb2Snes) can make the process seem impossible. After countless hours trying to compile and configure solutions, I've found a robust and surprisingly efficient way to get Annelid working with RetroArch using the sni bridge. Get ready, because in my experience, it works even better than on Windows! This detailed guide is for you if you use a Linux distribution (like Nobara/Fedora) and RetroArch with the bsnes-mercury core.

Prerequisites (Make sure you have these):

    Operating System: Linux (tested and working on Nobara 40 / Fedora 40).
    RetroArch (Flatpak): Version 1.19.0 (or newer) installed and running. Flatpak version is recommended for better compatibility.
    Core (Núcleo) "Nintendo - SNES / SFC (bsnes-mercury Performance)": Version v094 (Mercury Performance). This core is crucial for its support of the network commands we will use.
    Super Metroid ROM (NTSC/USA version): A functional ROM of the game, which must be loaded and running in RetroArch with the bsnes-mercury Performance core.
    Basic Linux Terminal Knowledge: Familiarity with commands like cd, ls, mkdir, mv, tar, chmod, sudo dnf install (or the equivalent for your distribution if you're not using Fedora/Nobara).
    Internet Connection: Necessary to download dependencies and sni binaries.

Step 0: Environment Preparation (Libraries and Tools)

Before you begin, make sure you have the essential tools on your system.

    Open Terminal: Open your terminal application (Konsole, GNOME Terminal, etc.).

    Install basic tools and libraries:
    Execute the following command in your terminal to install the necessary dependencies. This will ensure you have Git, compilers, and crucial libraries like libusb and libwebsockets:
    sudo dnf install git gcc make libusb1-devel libwebsockets-devel cmake -y
        git: For cloning repositories (although for sni we will use a direct binary download for simplicity).
        gcc, make, cmake: Essential tools for compiling software from source code, necessary for Annelid.
        libusb1-devel: Development library for USB communication.
        libwebsockets-devel: Development library for WebSocket communications, crucial for sni.

    Confirm the installation when dnf prompts you.

Part 1: Configure RetroArch for Network Commands

RetroArch must expose its memory through a network interface so that sni can access it.

    Start RetroArch: Open RetroArch.
    Load Game and Core:
        Ensure that the core "Nintendo - SNES / SFC (bsnes-mercury Performance)" is selected and loaded. The core version is v094 (Mercury Performance).
        Load your Super Metroid ROM.
        Start the game. The game must be running (or at least not closed) for RetroArch to properly listen for network commands.
    Enable Network Commands:
        From the RetroArch main menu, go to Settings.
        Navigate to Network.
        Look for the "Network Commands" option and ensure it is set to "On".
        Verify that the "Network Command Port" is configured to 55355.
        Ensure that "Stdin Commands" is set to "On".
    Save Configuration:
        To ensure these settings persist, go back to the RetroArch main menu, then to Configuration File and select "Save Current Configuration".
    Keep RetroArch open and the game running for the next steps.

Part 2: Install and Run the sni Bridge

sni (Super Nintendo Interface) is the tool that will act as a bridge. Annelid will connect to sni using the QUSB2SNES protocol, and sni in turn will read RetroArch's memory via network commands.

    Download the pre-compiled sni binary:
        Open your web browser and go to the official alttpo/sni "Releases" page: https://github.com/alttpo/sni/releases
        Look for the latest available version (at the time of this guide, it was v0.0.101).
        In the "Assets" section, download the file corresponding to your 64-bit Linux system. Look for the file ending in -linux-amd64.tar.xz (e.g., sni-v0.0.101-linux-amd64.tar.xz). This file will be downloaded to your ~/Descargas (Downloads) folder.

    Decompress and prepare sni:
        Open a new terminal (separate from the one you're using for RetroArch if you're running it from the terminal).
        Navigate to your Downloads folder: cd ~/Descargas
        Decompress the downloaded .tar.xz file. The tar -xJvf command is suitable for this format: tar -xJvf sni-v0.0.101-linux-amd64.tar.xz (Make sure to replace sni-v0.0.101-linux-amd64.tar.xz with the exact name of the file you downloaded). This will create a new directory (e.g., sni-v0.0.101-linux-amd64/) containing the sni executable.
        Create a dedicated directory in your home folder to store the sni binary: mkdir ~/sni_bridge
        Move the sni executable from the decompressed directory to the new ~/sni_bridge/ directory: mv sni-v0.0.101-linux-amd64/sni ~/sni_bridge/ (Verify the source directory name sni-v0.0.101-linux-amd64/ if decompression yielded a different name).
        Grant execution permissions to the sni binary so you can run it: chmod +x ~/sni_bridge/sni

    Run sni as a bridge:
        In the same terminal, navigate to the sni_bridge directory you created: cd ~/sni_bridge
        Execute sni, instructing it to connect to the RetroArch backend and the address where RetroArch is listening. It is essential that this terminal remains open and the sni process runs while you are using Annelid. ./sni retroarch --address localhost:55355
            ./sni: Executes the sni binary.
            retroarch: This is the specific subcommand to connect to RetroArch (not a flag argument like --backend).
            --address localhost:55355: Specifies the address (localhost) and port (55355) where RetroArch is listening for network commands.
            A successful output will show something like:

            2025/06/01 01:41:26.637246 sni-linux-amd64 v0.0.101 ...
            ...
            usb2snes: listening on 0.0.0.0:23074

            The line usb2snes: listening on 0.0.0.0:23074 confirms that sni is ready to receive connections from Annelid.

Part 3: Install and Run Annelid (Autosplitter)

Annelid is the autosplitter that will read game memory through the sni bridge.

    Obtain Annelid:
        If you don't have Annelid compiled yet, you'll need to get its source code and compile it. To do this, you need to install Rust and Cargo.
            Install Rust/Cargo: Follow the official instructions at: https://www.rust-lang.org/tools/install Using rustup is recommended for easy installation and updates.
            Download Annelid source code: Open a terminal and clone the repository (or download it as a ZIP if you prefer): git clone https://github.com/skishore/annelid.git
            Compile Annelid: Navigate to the Annelid directory and compile in "release" mode for better performance: cd annelid cargo build --release This will create the annelid executable inside ~/annelid/target/release/.

    Run Annelid:
        Open a third terminal (separate from RetroArch and sni).
        Navigate to the directory where the annelid executable is located: cd ~/annelid/target/release/
        Execute Annelid: ./annelid
        In the terminal output, you should see messages confirming successful connection and data reading:

        project_dirs = ProjectDirs { ... }
        Loading from "/home/jeetkundo/.config/annelid/settings.toml"
        Registering global hotkeys...registered
        Server version is "SNI-v0.0.101"
        Using device: ra://127.0.0.1:55355
        Connected.
        Infos {
            version: "1.21.0",
            dev_type: "retroarch",
            game: "Super",
            flags: [],
        }
        Split due to pickup

        The Connected. line indicates that Annelid connected to sni, and Split due to pickup confirms that it is reading game memory and detecting events (this latter message will appear once an in-game event triggers).

    Load Splits in Annelid:
        If Annelid has a graphical user interface (GUI) (as seen in the screenshot), use its menu to load your splits file (typically an .lss file exported from LiveSplit or a compatible format with Annelid).
        Configure the specific in-game events that should trigger each split (e.g., picking up an item, clearing a screen, etc.).

Part 4: Final Test and Verification

With all components configured and running, it's time for the crucial test.

    Verify that all three components are active:
        RetroArch: With Super Metroid loaded and running (or paused).
        sni: Running in its terminal, showing usb2snes: listening on 0.0.0.0:23074.
        Annelid: Running in its terminal, showing Connected. and your splits file loaded in its GUI.
    Start your Super Metroid speedrun.
    Perform the in-game actions you've configured to trigger your splits (e.g., first bomb, Varia Suit pickup, etc.).
    Observe Annelid: If everything is configured correctly, Annelid should detect these events and automatically perform the splits, updating the timer and current segment.

Congratulations! If the splits activate, you have successfully set up your autosplitter for Super Metroid on Linux. Enjoy your speedruns!


***************************************************************************************************************************
***************************************************************************************************************************
***************************************************************************************************************************

¡Éxito! Guía Completa: Autosplitter (Annelid) para Super Metroid en RetroArch Linux (Funciona Mejor que en Windows)

Esta guía paso a paso detalla cómo configurar Annelid (un autosplitter) para que funcione con RetroArch (utilizando el núcleo bsnes-mercury Performance) en un sistema operativo Linux (específicamente Nobara 40, basado en Fedora 40), permitiendo el autosplitting en speedruns de Super Metroid. Se utilizará el puente de software sni para la comunicación.

Objetivo: Habilitar la detección automática de eventos en el juego Super Metroid (a través de RetroArch) por parte de Annelid para automatizar los "splits" de tus speedruns.

Problema Común y Nuestra Solución:
¿Eres un speedrunner de Super Metroid en Linux y te has topado con la frustrante barrera de los autosplitters? La falta de guías claras y los problemas de compatibilidad (como los desafíos con la compilación de usbsnes o QUsb2Snes en Linux) pueden hacer que el proceso parezca imposible. Después de incontables horas intentando compilar y configurar soluciones, he encontrado una forma robusta y sorprendentemente eficiente de hacer funcionar Annelid con RetroArch utilizando el puente sni. ¡Prepárate, porque en mi experiencia, funciona incluso mejor que en Windows! Esta guía detallada es para ti si usas una distribución Linux (como Nobara/Fedora) y RetroArch con el núcleo bsnes-mercury.

Requisitos Previos (Verifica que tienes esto):

    Sistema Operativo: Linux (probado y funcionando en Nobara 40 / Fedora 40).
    RetroArch (Flatpak): Versión 1.21.0 instalada y funcionando. Se recomienda la versión Flatpak para evitar conflictos de librerías y facilitar la gestión.
    Core (Núcleo) "Nintendo - SNES / SFC (bsnes-mercury Performance)": Versión v094 (Mercury Performance). Este core es crucial por su soporte a los comandos de red necesarios para la comunicación.
    ROM de Super Metroid (versión NTSC/USA): Tu ROM funcional del juego, que debe cargarse y ejecutarse en RetroArch con el núcleo bsnes-mercury Performance.
    Conocimientos Básicos de Terminal en Linux: Familiaridad con comandos como cd, ls, mkdir, mv, tar, chmod, sudo dnf install (o el equivalente para tu distribución si no usas Fedora/Nobara).
    Conexión a Internet: Necesaria para descargar dependencias y los binarios de sni.

Paso 0: Preparación del Entorno (Librerías y Herramientas)

Antes de empezar, asegúrate de tener las herramientas esenciales en tu sistema.

    Abrir Terminal: Abre tu aplicación de terminal (Konsole, GNOME Terminal, etc.).

    Instalar herramientas básicas y librerías:
    Ejecuta el siguiente comando en tu terminal para instalar las dependencias necesarias. Esto asegurará que tengas Git, compiladores y librerías cruciales como libusb y libwebsockets:
    sudo dnf install git gcc make libusb1-devel libwebsockets-devel cmake -y
        git: Para clonar repositorios (aunque para sni usaremos una descarga directa del binario para mayor simplicidad).
        gcc, make, cmake: Herramientas esenciales para compilar software desde el código fuente, necesarias para Annelid.
        libusb1-devel: Librería de desarrollo para comunicación USB.
        libwebsockets-devel: Librería de desarrollo para comunicaciones WebSocket, crucial para sni.

    Confirma la instalación cuando dnf te lo pregunte.

Parte 1: Configurar RetroArch para Comandos de Red

RetroArch debe exponer su memoria a través de una interfaz de red para que sni pueda acceder a ella.

    Iniciar RetroArch: Abre RetroArch.
    Cargar Juego y Núcleo:
        Asegúrate de que el core "Nintendo - SNES / SFC (bsnes-mercury Performance)" esté seleccionado y cargado. La versión del núcleo es v094 (Mercury Performance).
        Carga tu ROM de Super Metroid.
        Inicia el juego. El juego debe estar corriendo (o al menos no cerrado) para que RetroArch escuche los comandos de red correctamente.
    Habilitar Comandos de Red:
        Desde el menú principal de RetroArch, ve a Ajustes (Settings).
        Navega a Red (Network).
        Busca la opción "Comandos de red" (Network Commands) y asegúrate de que esté en "Sí" (On).
        Verifica que el "Puerto de comandos de red" (Network Command Port) esté configurado en 55355.
        Asegúrate de que "Comandos stdin" (Stdin Commands) esté en "Sí" (On).
    Guardar Configuración:
        Para asegurar que estos ajustes persistan, ve de nuevo al menú principal de RetroArch, luego a Archivo de configuración (Configuration File) y selecciona "Guardar configuración actual" (Save Current Configuration).
    Mantén RetroArch abierto y el juego corriendo para los siguientes pasos.

Parte 2: Instalar y Ejecutar el Puente sni

sni (Super Nintendo Interface) es la herramienta que servirá como puente. Annelid se conectará a sni usando el protocolo QUSB2SNES, y sni a su vez leerá la memoria de RetroArch a través de los comandos de red.

    Descargar el binario precompilado de sni:
        Abre tu navegador web y dirígete a la página de "Releases" oficial de alttpo/sni: https://github.com/alttpo/sni/releases
        Busca la última versión disponible (en el momento de esta guía, era v0.0.101).
        En la sección "Assets", descarga el archivo que corresponde a tu sistema Linux de 64 bits. Busca el archivo que termina en -linux-amd64.tar.xz (ej., sni-v0.0.101-linux-amd64.tar.xz). Este archivo se descargará a tu carpeta ~/Descargas.

    Descomprimir y preparar sni:
        Abre una nueva terminal (diferente a la que usas para RetroArch si lo ejecutas desde terminal).
        Navega a tu carpeta de Descargas: cd ~/Descargas
        Descomprime el archivo .tar.xz descargado. El comando tar -xJvf es el adecuado para este formato: tar -xJvf sni-v0.0.101-linux-amd64.tar.xz (Asegúrate de reemplazar sni-v0.0.101-linux-amd64.tar.xz por el nombre exacto del archivo que descargaste). Esto creará un nuevo directorio (ej., sni-v0.0.101-linux-amd64/) que contendrá el ejecutable sni.
        Crea un directorio dedicado en tu carpeta personal para almacenar el binario sni: mkdir ~/sni_bridge
        Mueve el ejecutable sni desde el directorio descomprimido al nuevo directorio ~/sni_bridge/: mv sni-v0.0.101-linux-amd64/sni ~/sni_bridge/ (Verifica el nombre del directorio sni-v0.0.101-linux-amd64/ si la descompresión generó un nombre diferente).
        Asigna permisos de ejecución al binario sni para que puedas ejecutarlo: chmod +x ~/sni_bridge/sni

    Ejecutar sni como puente:
        En la misma terminal, navega al directorio sni_bridge que creaste: cd ~/sni_bridge
        Ejecuta sni, indicándole que se conecte al backend de RetroArch y la dirección donde RetroArch está escuchando. Es fundamental que esta terminal permanezca abierta y el proceso de sni en ejecución mientras uses Annelid. ./sni retroarch --address localhost:55355
            ./sni: Ejecuta el binario sni.
            retroarch: Este es el subcomando específico para conectar a RetroArch (no un argumento de bandera como --backend).
            --address localhost:55355: Especifica la dirección (localhost) y el puerto (55355) donde RetroArch está escuchando los comandos de red.
            Una salida exitosa mostrará algo como:

            2025/06/01 01:41:26.637246 sni-linux-amd64 v0.0.101 ...
            ...
            usb2snes: listening on 0.0.0.0:23074

            La línea usb2snes: listening on 0.0.0.0:23074 es la confirmación de que sni está listo para recibir conexiones de Annelid.

Parte 3: Instalar y Ejecutar Annelid (Autosplitter)

Annelid es el autosplitter que leerá la memoria del juego a través del puente sni.

    Obtener Annelid:
        Si aún no tienes Annelid compilado, deberás obtener su código fuente y compilarlo. Para ello, necesitas instalar Rust y Cargo.
            Instalar Rust/Cargo: Sigue las instrucciones oficiales en: https://www.rust-lang.org/tools/install Es recomendable usar rustup para una instalación sencilla y mantenerlo actualizado.
            Descargar código fuente de Annelid: Abre una terminal y clona el repositorio (o descárgalo como ZIP): git clone https://github.com/skishore/annelid.git
            Compilar Annelid: Navega al directorio de Annelid y compila en modo "release" para un mejor rendimiento: cd annelid cargo build --release Esto creará el ejecutable annelid dentro de ~/annelid/target/release/.

    Ejecutar Annelid:
        Abre una tercera terminal (distinta a las de RetroArch y sni).
        Navega al directorio donde se encuentra el ejecutable annelid: cd ~/annelid/target/release/
        Ejecuta Annelid: ./annelid
        En la salida de la terminal, deberías ver mensajes que confirman la conexión exitosa y la lectura de datos:

        project_dirs = ProjectDirs { ... }
        Loading from "/home/jeetkundo/.config/annelid/settings.toml"
        Registering global hotkeys...registered
        Server version is "SNI-v0.0.101"
        Using device: ra://127.0.0.1:55355
        Connected.
        Infos {
            version: "1.21.0",
            dev_type: "retroarch",
            game: "Super",
            flags: [],
        }
        Split due to pickup

        La línea Connected. indica que Annelid se conectó a sni, y Split due to pickup confirma que está leyendo la memoria del juego y detectando eventos (este último mensaje aparecerá una vez que se detecte un evento en el juego).

    Cargar Splits en Annelid:
        Si Annelid tiene una interfaz gráfica (GUI) (como se ve en la captura de pantalla), utiliza su menú para cargar tu archivo de splits (generalmente un archivo .lss exportado desde LiveSplit o un formato compatible con Annelid).
        Configura los eventos específicos del juego que deben activar cada split (por ejemplo, recoger un ítem, pasar una pantalla, etc.).

Parte 4: Prueba Final y Verificación

Con todos los componentes configurados y ejecutándose, es momento de la prueba crucial.

    Verifica que los tres componentes estén activos:
        RetroArch: Con Super Metroid cargado y en ejecución (o en pausa).
        sni: Ejecutándose en su terminal, mostrando usb2snes: listening on 0.0.0.0:23074.
        Annelid: Ejecutándose en su terminal, mostrando Connected. y tu archivo de splits cargado en su GUI.
    Comienza tu speedrun en Super Metroid.
    Realiza las acciones en el juego que has configurado para activar tus splits (por ejemplo, la primera bomba, la recolección de Varia, etc.).
    Observa Annelid: Si todo está configurado correctamente, Annelid debería detectar estos eventos y realizar los splits automáticamente, actualizando el cronómetro y el segmento actual.

¡Felicidades! Si los splits se activan, has configurado con éxito tu autosplitter para Super Metroid en Linux. ¡A disfrutar de tus speedruns!
