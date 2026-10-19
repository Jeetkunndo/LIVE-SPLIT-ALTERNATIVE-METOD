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


s speedruns!
