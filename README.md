# qvm-create-windows-qube

qvm-create-windows-qube is a tool for quickly and conveniently installing fresh new Windows [qubes](https://www.qubes-os.org) with Qubes Windows Tools as well as other packages such as Firefox, Office 365, Notepad++ and Visual Studio pre-installed modularly and automatically.

## Installation

1. Download the [installation script](https://raw.githubusercontent.com/elliotkillick/qvm-create-windows-qube/master/install.sh) by opening the link, right-clicking and then selecting "Save as..."
2. Copy `install.sh` into Dom0 by running the following command in Dom0: `qvm-run -p --filter-escape-chars --no-color-output <qube_script_is_located_on> "cat '/home/user/Downloads/install.sh'" > install.sh`
3. Review the code of `install.sh` to ensure its integrity
4. Run `chmod +x install.sh && ./install.sh`
5. Review the code of the resulting `qvm-create-windows-qube.sh`

## Usage

```
Usage: ./qvm-create-windows-qube.sh [options] <name>
  -h, --help
  -c, --count <number> Number of Windows qubes with given basename desired
  -t, --template Make this qube a TemplateVM instead of a StandaloneVM
  -n, --netvm <qube> NetVM for Windows to use
  -s, --seamless Enable seamless mode persistently across restarts
  -o, --optimize Optimize Windows by disabling unnecessary functionality for a qube
  -y, --anti-spy Disable Windows telemetry
  -p, --packages <packages> Comma-separated list of packages to pre-install (see available packages at: https://chocolatey.org/packages)
  -i, --iso <file> Windows media to automatically install and setup (default: win7x64-ultimate.iso)
  -a, --answer-file <xml file> Settings for Windows installation (default: win7x64-ultimate.xml)
```

Example: `./qvm-create-windows-qube.sh -n sys-firewall -soyp firefox,notepadplusplus,office365business win7-work`

Please see QWT Known Issues before continuing.

Note: If the Qubes GUI driver is unwanted, either due to stability or peformance issues, then that can be disabled by going into the answer file and removing everything under the RunSynchronus tag for enabling test signing. Make sure you also cause a new ISO to be generated by deleting the old one in the out folder. This works because the GUI driver is the only unsigned driver and the QWT installer will automatically not install the GUI driver if it detects test signing is disabled.

## Security

The resources qube, windows-mgmt by default, is air gapped. To mitigate the fallout of another [Shellshock](https://en.wikipedia.org/wiki/Shellshock_(software_bug)) Bash vulnerability, the Dom0 script communicates to the windows-mgmt qube in a one-way fashion. Downloading of the Windows ISOs are made secure by encforcing TLS 1.2/1.3 HTTPS with HTTP public key pinning (HPKP) and by verifying the SHA-256 of the files after download. Packages such as Firefox are offered out of the box so the infamously insecure Internet Explorer never has to be used.

Windows 7/Windows Server 2008 R2 will reach [End Of Life (EOL) on January 14, 2020](https://support.microsoft.com/en-us/help/4057281/windows-7-support-will-end-on-january-14-2020). However, Office 365 for these operating systems will continue getting security updates until [January 2023](https://support.office.com/en-us/article/windows-7-end-of-support-and-office-78f20fab-b57b-44d7-8368-06a8493f3cb9).

Important: If RDP is to be enabled on the Windows qube (not default) then make sure it is fully up-to-date because the latest Windows 7 ISO Microsoft offers is unfortunately still vulnerable to [BlueKeep](https://en.wikipedia.org/wiki/BlueKeep) and related DejaBlue vulnerabilites

## Privacy

Privacy benefits by disabling unwanted Microsoft telemetry such as the Customer Experience Improvement Program (CEIP), Windows Error Reporting (WER) and Diagnostics Tracking service (DiagTrack), standardizing common Whonix recommeneded defaults such as "user" for the username and "host" for the hostname, and by resetting unique identifiers present in every Windows installation such as the MachineGUID, NTFS drive Volume Serial Numbers (VSNs) and more.

However, there are still ways to fingerprint you through the hypervisor (not specific to Windows): [lscpu](https://github.com/QubesOS/qubes-issues/issues/1142), [timezone](https://github.com/QubesOS/qubes-issues/issues/4429) (Can be mitigated by configuring UTC time in the BIOS/UEFI), screen resolution and depth, generally some of the VM interfaces documented [here](https://www.qubes-os.org/doc/vm-interface), as well as much more obscure things.

## Contributing

You can start by giving this project a star! PRs are also welcome! Take a look at the todo list below if you're looking for things that need improvement. Other improvements such as more elegant ways of completing a task, code cleanup and other fixes are also welcome. :)

Lots of Windows-related [GSoCs](https://www.qubes-os.org/gsoc) for those interested.

Note: This project is the product of an independent effort that is not offically endorsed by Qubes OS

## QWT Known Issues

HVMs, not QWT issue (Windows, sys-net, etc.):
- Until this [bug](https://github.com/QubesOS/qubes-issues/issues/4684) is fixed upstream you must fix app menu syncing for HVMs (Windows, sys-net, etc.) by putting the provided `qubes-start.desktop` into `/usr/share/qubes-appmenus` in Dom0

All OSs:
- Windows may crash on first boot after QWT is installed
    - On Windows 7, the desktop may boot back up to having no wallpaper and missing libraries. Just delete that qube and start over
    - Or it may crash during app menu creation shortly after first boot after desktop wallpaper/libraries is setup (qvm-sync-appmenus <qube_name> will fix)
    - Or it might just hang on trying to copy the post scripts over to the crashed qube
    - Fix: Just delete the qube and start over

All OSs except Windows 7:
- Run `qvm-features <windows_qube> gui 1` to make display show up after setup is complete
- When Qubes GUI driver is in use, may receive a message saying Windows is trying to spoof Dom0 GUI causing installation to pause. Fix by closing window
- Prompt to install earlier version of .NET (However, qrexec services still seem to work): https://github.com/QubesOS/qubes-issues/issues/5091 (Waiting for this to make it into a QWT update, has been merged)
- No GUI driver yet
    - The resolution can still be increased to 1920x1080 or higher by increasing the display resolution in Windows

Windows 10/Windows Server 2019:
- Private disk creation fails: https://github.com/QubesOS/qubes-issues/issues/5090 (Waiting for this to make it into a QWT update, has been merged)
    - Temp fix: Close prepare-volume.exe window causing there to be no private disk (can't make a template VM) but besides that it will continue as normal

See here:

https://github.com/QubesOS/qubes-issues/labels/C%3A%20windows-tools
https://github.com/QubesOS/qubes-issues/labels/C%3A%20windows-vm

## Todo

- [x] Gain the ability to reliably unpack/insert answer file/repack for any given ISO 9660 (Windows ISO format)
    - Blocking issue for supporting other versions of Windows
- [x] auto-qwt takes D:\\ making QWT put the user profile on E:\\; it would be nicer to have it on D:\\ so there is no awkward gap in the middle
- [x] Support Windows 8.1-10 (Note: QWT doesn't fully offically any OS other than Windows 7 yet, however, everything is functional except the GUI driver)
- [x] Support Windows Server 2008 R2 to Windows Server 2019
- [x] Support Windows 10 Enterprise LSTC (Long Term Support Channel, provides security updates for 10 years, very stable and less bloat than stock Windows 10)
- [x] Provision Chocolatey
- [x] Add an option to slim down Windows as documented for Qubes [here](https://www.qubes-os.org/doc/windows-template-customization/)
- [x] Make windows-mgmt air gapped
- [ ] Possibly switch from udisksctl for reading/mounting ISOs because it is written in its man page that it is not intended for scripts
    - guestfs
    - losetup/mount (requires sudo, but it's what's used by the Qubes Core Team in their scripts)
    - Consider other alternatives
- [ ] I recently disovered this is a Qubes [Google Summer of Code](https://www.qubes-os.org/gsoc) project; which is cool
    - [x] Add automated tests
        - Using Travis CI for automated ShellCheck
    - [ ] ACPI tables for fetching Windows the license embedded there
        - It mentions use of C, however, it seems like it may be possible to [just use shell](https://osxdaily.com/2018/09/09/how-find-windows-product-key)
    - [ ] Port to Python
        - This seems like it would only add unnecessary LOC to scripts like create-media.sh where the Python script would essentially just be calling losetup/udisksctl and genisoimage
        - This would certainly be suitable for qvm-create-windows-qube.sh though
            - This would allow us to interchange data between Dom0 and the VM without worring about another Shellshock
- [ ] Use wiminfo command to identify what version and editions of Windows an ISO contains
    - Works just like DISM on Windows
    - As simple as 1. Mount ISO 2. wiminfo install.wim (Parses XML into neat list)

## End Goal

Have feature similar (or superior) to [VMWare's Windows "Easy Install"](https://www.youtube.com/watch?v=1OpDXlttmE0) feature on Qubes.
