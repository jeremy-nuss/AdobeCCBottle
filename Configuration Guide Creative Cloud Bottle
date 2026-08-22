# Adobe Premiere Pro 2026 on Linux Guide

Running modern Adobe applications on Linux requires an advanced set of workarounds due to heavy reliance on undocumented Windows APIs, missing DirectX 2D capabilities in Wine, and CPU checks. This guide outlines the exact steps to get Premiere Pro 2026 running, with video playback working and the UI rendering reliably in software mode.

> [!WARNING]
> This guide disables hardware acceleration for the Premiere Pro interface (Drawbot). This is required to bypass a missing Direct2D feature in Wine (`E_NOTIMPL`) that causes black panels and crashing. Video playback itself remains functional.

---

## 0. Prerequisites

Before starting, ensure you have the necessary tools installed on your Linux system.

### Install Winetricks
Winetricks is required to fetch native Microsoft DLLs and fonts that Adobe relies on.
```bash
# Download the latest winetricks script
wget  https://raw.githubusercontent.com/Winetricks/winetricks/master/src/winetricks
chmod +x winetricks
sudo mv winetricks /usr/local/bin/
```

### Get a Custom Runner (proton-10.0-adobe)
Adobe apps rarely work on standard Wine. You need a custom runner optimized for Adobe, specifically the `proton-10.0-adobe` runner.

**Option A: Via Bottles GUI**
1. Open **Bottles**.
2. Click the hamburger menu (three lines) in the top left and select **Preferences**.
3. Under the **Runners** section, locate and download the `proton-10.0-adobe` runner (if available in your repository).

**Option B: Via Terminal (If the Runners screen is blank or missing the custom runner)**
If Bottles fails to load the list of runners or you are using an unofficial Adobe build, you can manually download and extract it into the Bottles directory:
```bash
# Create the runners directory if it doesn't exist
mkdir -p ~/.local/share/bottles/runners/

# Download the proton-10.0-adobe runner
wget https://github.com/PhialsBasement/wine-adobe-installers/releases/download/proton-10.0-adobe/proton-steamcompattool-adobe-linux-x86_64.tar.xz

# Extract it into the bottles runner directory
tar -xf proton-steamcompattool-adobe-linux-x86_64.tar.xz -C ~/.local/share/bottles/runners/

# Clean up the archive
rm proton-steamcompattool-adobe-linux-x86_64.tar.xz
```
*(Restart Bottles after doing this, and "proton-10.0-adobe" will appear in the Runner dropdown for your bottle).*

## 1. Initial Bottles Setup

You will use **Bottles** (a GUI wrapper for Wine) to manage the prefix.

1. Open Bottles and create a new **Application** bottle. Name it `AdobeCC`.
2. Open the Settings for `AdobeCC` and configure the following:
   *   **Runner:** Use your installed custom runner (`proton-10.0-adobe`).
   *   **Windows Version:** Windows 10
   *   **Virtual Desktop:** Enabled (Set Resolution to `1920x1080` or your monitor's resolution). 
       *To find this in Bottles: Go to your Bottle -> **Settings** -> **Display** -> Toggle **Virtual Desktop**.*
       *(Fallback method: If you cannot find this in Bottles, click **Legacy Tools** -> **Wine Configuration** -> **Graphics** tab -> Check "**Emulate a virtual desktop**" and set the resolution).*
       > [!NOTE] 
       > Premiere uses custom "owner-drawn" window borders. If you do not use a Virtual Desktop, your Linux window manager may force Premiere into a maximized state that cannot be resized.

## 2. Install Dependencies

Before installing Creative Cloud, you must inject native Windows fonts and libraries to support Adobe's login services.

> [!WARNING]
> Do **NOT** use the Dependencies menu inside the Bottles GUI to install `msxml3`. The Bottles downloader for `msxml3` is currently broken and will cause the Creative Cloud installer to crash. You MUST use `winetricks` via the terminal.

Open your terminal and run the following command to install the required Windows components directly into your bottle (if you named your bottle something other than `AdobeCC`, update the path accordingly):

```bash
WINEPREFIX=~/.local/share/bottles/bottles/AdobeCC WINE=~/.local/share/bottles/runners/proton-10.0-adobe/bin/wine winetricks -q vcrun2022 msxml3 msxml6 corefonts gdiplus
```
*(Note: You can safely ignore any "warning: You are using a 64-bit WINEPREFIX" messages during this process).*

## 3. Install Microsoft Edge WebView2
The Adobe Creative Cloud installer requires Microsoft Edge WebView2 to render its login and setup interface. Without it, you will see a blank white screen or the installer will silently crash.

1. Download the **Offline Standalone Installer** for Edge WebView2 from Microsoft (Do not use the bootstrapper):
```bash
wget "https://go.microsoft.com/fwlink/p/?LinkId=2124701" -O ~/Downloads/MicrosoftEdgeWebView2RuntimeInstallerX64.exe
```
2. Run the offline installer silently inside your bottle:
```bash
WINEPREFIX=~/.local/share/bottles/bottles/AdobeCC WINE=~/.local/share/bottles/runners/proton-10.0-adobe/bin/wine ~/.local/share/bottles/runners/proton-10.0-adobe/bin/wine ~/Downloads/MicrosoftEdgeWebView2RuntimeInstallerX64.exe /silent /install
```
*(Wait a minute or two for this command to finish and return you to the prompt. It installs in the background).*

## 4. Patching the Windows Version (Build Number)
Adobe Creative Cloud 2026 will refuse to install or launch if it detects an outdated build of Windows 10. Wine's default Windows 10 build number is often too old. You must update the registry to trick Adobe into seeing a modern Windows 11 build (e.g., 23H2 / 22631) *before* you run the installer.

1. Create a script named `update_os.reg` on your Linux desktop with the following code:
```registry
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\Software\WOW6432Node\Microsoft\Windows NT\CurrentVersion]
"CurrentBuild"="22631"
"CurrentBuildNumber"="22631"
"BuildLab"="22631.ni_release.220506-1250"
"BuildLabEx"="22631.1.amd64fre.ni_release.220506-1250"
"DisplayVersion"="23H2"
"ReleaseId"="23H2"
"UBR"=dword:00000b2b

[HKEY_LOCAL_MACHINE\Software\Microsoft\Windows NT\CurrentVersion]
"CurrentBuild"="22631"
"CurrentBuildNumber"="22631"
"BuildLab"="22631.ni_release.220506-1250"
"BuildLabEx"="22631.1.amd64fre.ni_release.220506-1250"
"DisplayVersion"="23H2"
"ReleaseId"="23H2"
"UBR"=dword:00000b2b
```
2. Open Bottles, go to your `AdobeCC` bottle, click **Legacy Tools**, and select **Registry Editor**.
3. In the Registry Editor, click **Registry** (top left) -> **Import Registry File...**
4. Select your `update_os.reg` file to inject it. Close the Registry Editor.

## 4. Environment Variables

Newer processors (like Intel 10th Gen+ and newer Ryzen CPUs) have specific instruction sets that cause Adobe's web-based installers to instantly crash under Wine. We must disable these.

In Bottles, go to **Settings > Environment Variables** and add the following:

| Variable | Value | Purpose |
| :--- | :--- | :--- |
| `JSIMD_FORCENONE` | `1` | Disables broken SIMD instructions in libjpeg for Wine |
| `OPENSSL_ia32cap` | `~0x200000200000000` | Prevents the Creative Cloud installer from crashing on newer CPUs by disabling SHA extensions |

## 5. DLL Overrides

In Bottles, go to **Settings > DLL Overrides** (or open `winecfg` > Libraries) and ensure the following are set:

*   `msxml3` -> **Native, Builtin** *(Installed by winetricks)*
*   `gdiplus` -> **Native, Builtin** *(Installed by winetricks)*

If you are forcing Software Rendering for the UI later in this guide, you do *not* need to disable `d3d11` or `dxgi`. Keep DXVK enabled in Bottles so your video timeline can still playback smoothly.

## 6. Install Creative Cloud & Premiere Pro

1. Run the `Creative_Cloud_Set-Up.exe` installer inside your `AdobeCC` bottle.
2. Complete the installation and sign in.
3. Once Creative Cloud is open, install **Adobe Premiere Pro**.
4. Do **not** launch Premiere Pro yet. Close Creative Cloud entirely.

## 7. Patching the CPU Instruction Crash

Premiere Pro 2026 contains a bug in `jpeg_wrapper.dll` that attempts to execute instructions at a null memory address when initializing image processing on Wine, causing a fatal hard crash on the splash screen.

You must hex-patch the DLL to remove these instructions.

1. Create a script named `patch_jpeg.py` on your Linux desktop with the following code:
```python
import os
import shutil

# Update this path to match your actual Bottle location and username
dll_path = os.path.expanduser("~/.local/share/bottles/bottles/AdobeCC/drive_c/Program Files/Adobe/Adobe Premiere Pro 2026/jpeg_wrapper.dll")
backup_path = dll_path + ".bak"

if not os.path.exists(backup_path):
    shutil.copy2(dll_path, backup_path)
    print("Backup created.")

with open(dll_path, "r+b") as f:
    # NOP the mov instructions that dereference the null pointer
    f.seek(0x491bb7)
    f.write(b"\x90" * 7)
    
    # NOP the call instruction that executes the null pointer
    f.seek(0x491be0)
    f.write(b"\x90" * 7)

print("Successfully patched jpeg_wrapper.dll!")
```
2. Run the script: `python3 patch_jpeg.py`

## 8. Fixing UI Glitches (The Drawbot Error)

Premiere Pro's "Drawbot" UI engine heavily relies on Direct2D. Because Wine's `d2d1.dll` is missing several advanced functions (throwing an `E_NOTIMPL` / `-2147467263` error), the Timeline and Project Bins will render completely black or glitch out. 

You must force Premiere Pro to abandon DirectX UI rendering and fall back to pure Software GDI. We do this by injecting hidden flags into Adobe's Debug Database.

1. Open a terminal and run the following script (save it as `update_debug.py`):
```python
import re
import os

# Update this path to match your username (e.g. steamuser, or jeremy)
db_path = os.path.expanduser("~/.local/share/bottles/bottles/AdobeCC/drive_c/users/steamuser/AppData/Roaming/Adobe/Premiere Pro/26.0/Debug Database.txt")

if not os.path.exists(db_path):
    print("Debug Database not found! Please run Premiere Pro once so it generates the file, then close it.")
    exit(1)

with open(db_path, 'r') as f:
    content = f.read()

# Force pure software rendering for the UI
keys = [
    ("Droste.EnableHardwareAcceleration", "false"),
    ("Frontend.EnableHardwareRendering", "false"),
    ("drawbot.UseSoftwareRenderer", "true"),
    ("drawbot.Direct2D.enableDebugLayer", "false"),
    ("DS.DisableDirectXDisplay", "true"),
    ("Player.DirectXAcceleratedRenderer", "false"),
    ("GF.DisableAcceleratedRenderer", "true"),
]

for key, val in keys:
    content = re.sub(rf'{key}\t.*\n', '', content)

with open(db_path, 'w') as f:
    f.write(content)
    for key, val in keys:
        f.write(f'{key}\t{val}\t{val}\n')

print("Debug Database updated.")
```
2. Run the script: `python3 update_debug.py`

> [!TIP]
> If the `Debug Database.txt` file does not exist, you must launch Premiere Pro at least once (let it crash or freeze), then close it. Adobe generates this file on the first run.

## 9. Launching the Application

> [!IMPORTANT]
> **Always launch Premiere Pro using the Play button inside the Bottles GUI.**
> 
> If you launch the `.exe` or a `.bat` file directly from a Linux command prompt, Bottles will **not** inject the Environment Variables, DLL Overrides, or the Virtual Desktop. This will cause the application to crash immediately or corrupt the UI.

1. Open the Bottles GUI.
2. Select your `AdobeCC` bottle.
3. Click the Play button next to **Adobe Premiere Pro**.
4. The application will launch inside a resizable Virtual Desktop window with a working timeline and video playback!
