# Windows Tools

## Package Managers
 - [Chocolatey](https://chocolatey.org/)
   a console based package manager for Windows (it also has a GUI if desired)  
   Most of the following tools can be installed via Chocolatey

### Chocolatey

Install: https://chocolatey.org/install

Chocolatey programs:

```powershell
choco install vscode --params "/NoDesktopIcon /NoQuicklaunchIcon"
choco install ditto
choco install 7zip
choco install linqpad
choco install microsoft-windows-terminal
choco install docker-desktop
choco install sourcetree
choco install everything --params "/start-menu-shortcuts /run-on-system-startup"
choco install paint.net
choco install vlc
choco install beyondcompare
choco install poshgit
choco install powershell-core
choco install screentogif
```

- List of programs installed by chocolatey: `choco list --localonly` 
- Upgrade all programs: `choco upgrade all`


## Disk Size
 - [WizTree](https://antibody-software.com/web/software/software/wiztree-finds-the-files-and-folders-using-the-most-disk-space-on-your-hard-drive/) - recommended
 - WinDirStat
 - TreeSize


## Version Control
 - [SourceTree](https://www.sourcetreeapp.com/) (GIT)


## File Searching
 - [Everything](https://www.voidtools.com/)


## Hardware Monitoring
 - [Open Hardware Monitor](https://openhardwaremonitor.org/)  
   A no install hardware monitoring software, useful for checking CPU temperature.


## Media Players
 - [VLC Media Player](https://www.videolan.org/)
 - [FastStone Image Viewer](https://www.faststone.org/FSViewerDetail.htm)


## Editors
 - [Paint.Net](https://www.getpaint.net/)
 - [ScreenToGif](https://www.screentogif.com/)
   Tool for recording your screen to a gif.


## IP Scanning
 - [Angry IP Sanner](https://angryip.org)


## Compression
 - [7zip](https://www.7-zip.org/)


## Web Debugging


## Debugging Proxy Server
 - [Fiddler](https://www.telerik.com/fiddler/fiddler-classic)
 - [Charles](https://www.charlesproxy.com)


## API Client
 - [PostMan](https://www.postman.com)
 - [Insomnia](https://insomnia.rest)


## Mouse

### Evoluent VerticalMouse 4

There is an install for the support software on chocolatey, but it is an old version. So will use the version from the website: https://evoluent.com/support/download/

## Audio

### VB-Cable

Website is here: https://vb-audio.com/Cable/

VB-Cable provides a virtual audio input and output that are linked. This allows the output of one program to be provided as the input for another.
For microphones this is useful if programs are auto adjusting the microphone's gain (quite a few programs do this). This can be problematic if the microphone is sensitive and you want its gain to remain fixed.

Do the following to setup VB-Cable and reroute microphone audio:
1. Download VBCABLE and extract.
2. Run VBCABLE_Setup_x64.exe in administrator mode.
3. Click **Install Driver**.
4. Go to **Control Panel > Sound** to see the list of playback and recording devices
5. Go to the Recording tab and open the properties of desired input microphone
   1. Go to the **Advanced** tab and note down the audio details under **Default Format**
   2. Go to the **Listen** tab and check **Listen to this device**
   3. Change **Playback through this device** to `CABLE Input (VB-Audio Virtual Cable)`.
   4. Click **Apply**, then **OK**.
6. Open the `CABLE Output` device's properties
   1. Go to the Advanced tab and ensure the Default Format matches the microphone.
   2. Set `CABLE Output` as the default recording device.
8. Go to the Playback tab and Open the `CABLE Input` device's properties, go to the Advanced tab and ensure the Default Format matches the microphone.
9. Go to the VB-Cable extracted folder and run VBCABLE_ControlPanel.exe
   1. Check that the Input and Output are the same and match the microphone's specs (they should based on the above steps)
   2. Follow the instructions here to set the Max Latency: https://vb-audio.com/Cable/VBCABLE_SystemSettings.pdf
      For a 48000Hz microphone and an Internal SR of 96000Hz, then the Max Latency should be set to either 3072 or 4096 smp (I opted for 4096).
10. Restart the computer.
11. Shift the VBCABLE directory to a suitable location to access again if or when needed.

These changes should stop programs changing the gain on your microphone.

To uninstall rerun VBCABLE_Setup_x64.exe in administrator mode, and click **Remove Drivers**. Then restart the computer.

**Trouble Shooting**
If the virtual audio cable stops working see if the cable has been muted:
1. Go into Sound.
1. Go to the **Playback** tab.
1. Go to CABLE Input **Properties**.
1. Go to the **Levels** tab.
1. Check CABLE Input is not muted.
1. Ensure the input level is 100.

Repeat the same steps for CABLE Output under the Recording tab.

---

Or try unplugging the mic and replugging it in.

Another option to check is that the mic proper still has **Listen to this device** checked. And **Playback through this device** is still set to the CABLE input device. This is under the mic's Listen tab.


