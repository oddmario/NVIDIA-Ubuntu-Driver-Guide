# NVIDIA Ubuntu Driver Guide
A little guide to help you install & manage the NVIDIA GPU driver on your Ubuntu system(s)

When I wrote this guide, I was personally using **Ubuntu 24.04.1**, so this is mostly what this guide applies to (though I believe it should work alright on newer releases, and also on older releases which are not old very old `[something like Ubuntu 20.04+]`)

## Index of content
- [Driver installation](#driver-installation)
  * ⭐ [Through the graphics-drivers PPA repository](#-installing-through-the-graphics-drivers-ppa-repository-recommended) [RECOMMENDED]
  * [Through the official NVIDIA installer from the Nvidia.com website](#installing-through-the-official-nvidia-installer-from-the-nvidiacom-website)
- [Driver uninstallation](#driver-uninstallation)
  * [When installed through the graphics-drivers PPA repository](#uninstalling-the-driver-when-installed-through-the-graphics-drivers-ppa-repository)
  * [When installed through the official NVIDIA installer from the Nvidia.com website](#uninstalling-the-driver-when-installed-through-the-official-nvidia-installer-from-the-nvidiacom-website)
- [Issues faced after installing the NVIDIA drivers, and how to solve them](#issues-faced-after-installing-the-nvidia-drivers-and-how-to-solve-them)
  * [Fix ghost "Unknown Display" issue](#theres-a-ghost-unknown-display-on-the-gnome-displays-settings-especially-if-you-followed-the-graphics-drivers-ppa-repository-installation-procedure)
  * [Wayland is no longer enabled/not visible on the login screen](#wayland-is-not-shown-as-an-option-on-the-login-screen-or-the-cog-icon-of-the-login-screen-doesnt-show-at-all)
  * [Fix Wayland issues (flickering, etc.)](#the-experience-on-wayland-is-not-the-smoothest-fix-wayland-issues)
- [References](#references)

-----

> ## ⚠️ Warning
> 
> Please follow & read every part of this guide with fine care to avoid the occurrence of any problems.
> 
> Also do not worry if the system looks stuck during any rebooting step. It actually is not stuck! Kindly allow up to 2 minutes for the rebooting to complete.

## Driver installation

### ⭐ Installing through the `graphics-drivers` PPA repository [RECOMMENDED]

This installation approach is the recommended one, and it actually is the method that I use to maintain an installation of the driver on my own system(s).

1. Ensure that you have uninstalled any previously installed NVIDIA drivers:
   * to uninstall any Nvidia drivers installed from an APT repository:
      ```
      sudo apt-get remove --purge '^nvidia-.*'
      sudo apt autoremove
      sudo apt install ubuntu-drivers-common -y
      reboot
      ```
   * to uninstall any Nvidia drivers installed using the official NVIDIA installer (`.run` file): [Driver uninstallation](#uninstalling-the-driver-when-installed-through-the-official-nvidia-installer-from-the-nvidiacom-website)
   * after rebooting, you may need to enter text-only mode (Ctrl + Alt + F3) before continuing.
2. Install these dependencies to ensure DKMS support and a problem-free installation process:
```
sudo apt install pkg-config libglvnd-dev dkms build-essential libegl-dev libegl1 libgl-dev libgl1 libgles-dev libgles1 libglvnd-core-dev libglx-dev libopengl-dev gcc make 
```

3. Add the repository and install the driver:
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update 
sudo apt install nvidia-driver-570
reboot
```

NOTE: At the time this guide was last updated, 570 is the latest tested version of the driver available on the repository.

Navigate to https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa to check what the latest version of the driver is, then replace the `driver-570` part with the version you would like to install.

4. Once the system has rebooted, run `nvidia-smi` to confirm that the driver has been installed with no issues.

### Installing through the official NVIDIA installer from the Nvidia.com website

This procedure is more advanced and is often not recommended. However, it shall go alright as long as you follow each step with patience and care :)

1. Ensure that you have uninstalled any previously installed NVIDIA drivers:
   * to uninstall any Nvidia drivers installed from an APT repository:
      ```
      sudo apt-get remove --purge '^nvidia-.*'
      sudo apt purge libnvidia-*
      sudo apt autoremove
      sudo apt install ubuntu-drivers-common -y
      reboot
      ```
   * to uninstall any Nvidia drivers installed using the official NVIDIA installer (`.run` file): [Driver uninstallation](#uninstalling-the-driver-when-installed-through-the-official-nvidia-installer-from-the-nvidiacom-website)

2. Ensure that you do not have a manually installed version of `libnvidia-egl-wayland1` (especially if you are going to install version 555+ of the Nvidia driver). The driver already includes it as stated @ https://us.download.nvidia.com/XFree86/Linux-x86_64/555.42.02/README/installedcomponents.html
```
sudo apt remove libnvidia-egl-wayland1
```

3. Install the required dependencies:
```
sudo apt install pkg-config libglvnd-dev dkms build-essential libegl-dev libegl1 libgl-dev libgl1 libgles-dev libgles1 libglvnd-core-dev libglx-dev libopengl-dev gcc make 
```

4. Navigate to https://www.nvidia.com/Download/index.aspx?lang=en-us and download the proper driver for your GPU and Linux architecture. The website should give you a file that ends with the `.run` file extension.

5. Switch to the terminal view of your system by pressing `Ctrl + Alt + F3` (if this does not switch from the GUI mode to the terminal mode for you, try `Ctrl + Alt + F1` or `Ctrl + Alt + F2` instead for a different tty)

6. Stop the GDM service:
```
sudo systemctl stop gdm
sudo systemctl stop gdm3
```
If this fails for you, try `sudo systemctl stop lightdm` instead.

**Kindly note** that it is important to stop the GNOME Display Manager (GDM) service throughout the driver installation/uninstallation process as it may cause trouble otherwise.

7. Change to the path of the directory that includes the downloaded `.run` file using `cd`

8. Run the installer:
```
chmod +x NVIDIA-Linux-x86_64-555.42.02.run
sudo sh ./NVIDIA-Linux-x86_64-555.42.02.run
```
(make sure to replace the file name with the actual one that you got from the Nvidia website)

9. The installer will guide you through everything. Please read everything with care and answer the prompts depending on the proper situation to avoid any problems.
   
NOTE: If the installer asks you to disable Nouveau, allow the installer to disable it for you. You may need to abort the installer after this, then run `sudo update-initramfs -u && reboot`, then start again from step 5 once the system has completed rebooting.

10. Once the installer has completed installing the driver, run `sudo update-initramfs -u` to update the initramfs.
11. Edit `/etc/default/grub` using `sudo nano /etc/default/grub`
12. Add `nvidia-drm.modeset=1` and `nvidia-drm.fbdev=1` inside your `GRUB_CMDLINE_LINUX` (i.e. `GRUB_CMDLINE_LINUX="nvidia-drm.modeset=1 nvidia-drm.fbdev=1"`)
13. Run `sudo update-grub`
14. Reboot the system
15. Your newly installed driver should be up and running once the system boots up (you may run `nvidia-smi` to confirm so).

-----

## Driver uninstallation

### Uninstalling the driver when installed through the `graphics-drivers` PPA repository

Run:
```
sudo apt-get remove --purge '^nvidia-.*'
sudo apt autoremove
sudo apt install ubuntu-drivers-common -y
reboot
```

### Uninstalling the driver when installed through the official NVIDIA installer from the Nvidia.com website

1. To ensure that we can boot into the system graphically through the Nouveau driver after uninstalling the Nvidia driver, remove any Nouveau-blacklist entries that might have been created by the installer previously:
```
sudo rm -rf /lib/modprobe.d/nvidia-installer-*
sudo rm -rf /etc/modprobe.d/nvidia-installer-*
sudo rm -rf /usr/lib/modprobe.d/nvidia-installer-*
```
2. Remove any entries related to the NVIDIA driver (`nvidia-drm.modeset`, `nvidia-drm.fbdev`, etc) from your `/etc/default/grub` file. (__this is important__).
3. Rebuild the GRUB configuration using `sudo update-grub`
4. Run the uninstaller:
```
sudo nvidia-installer --uninstall
```
5. Rebuild the system initramfs:
```
sudo update-initramfs -u
```
6. Reboot the system once the uninstalling process has finished.

-----

## Issues faced after installing the NVIDIA drivers, and how to solve them

### There's a ghost "Unknown Display" on the GNOME Displays settings (especially if you followed the `graphics-drivers` PPA repository installation procedure).
  
This seems to be a bug reported at https://bugs.launchpad.net/ubuntu/+source/nvidia-graphics-drivers-535/+bug/2063222

A workaround is:
```
[ Workaround ]

1. sudo rm /dev/dri/card0
2. Log in again.
```

### Wayland is not shown as an option on the login screen (or the cog icon of the login screen doesn't show at all)

1. Edit the `/etc/gdm3/custom.conf` file using `sudo nano /etc/gdm3/custom.conf`
2. Ensure that `WaylandEnable=true` is set in that file and make sure that it's uncommented (does not start with a `#`)
3. Run `sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules`
4. Reboot the system

### The experience on Wayland is not the smoothest (fix Wayland issues)

This may happen for a lot of reasons. For a while now, NVIDIA has been known to have issues with the Wayland windowing system. However, NVIDIA has been working on making this better.
And this has actually already gotten much better starting from the NVIDIA driver 555.42.02 which added [explicit sync](https://9to5linux.com/developer-explains-why-explicit-sync-will-finally-solve-the-nvidia-wayland-issues) support.

So first of all, make sure to have:
- Version 555.42.02 or a higher version of the Nvidia driver
- GNOME 46.1 or a higher version on your Ubuntu installation

then continue reading below to make the experience even smoother:

* Your system may be using the Mesa driver instead of the NVIDIA one on Wayland sessions. You can confirm this by typing `glxinfo | egrep "OpenGL vendor|OpenGL renderer*"`

  In order to solve this:
     
  1. Edit `/etc/default/grub` using `sudo nano /etc/default/grub`
  2. Add `nvidia-drm.modeset=1` and `nvidia-drm.fbdev=1` inside your `GRUB_CMDLINE_LINUX` (i.e. `GRUB_CMDLINE_LINUX="nvidia-drm.modeset=1 nvidia-drm.fbdev=1"`)
  3. Run `sudo update-grub`
  4. Reboot the system
 
* You may have the GSP firmware of Nvidia enabled, and this is known to cause some performance issues on the beta 555.42.02 version of the driver. Maybe this will be fixed in the future, but for now, we can disable the GSP firmware if needed.

  You can check whether the GSP firmware is enabled or no by typing `nvidia-smi -q | grep "GSP Firmware"` — if it says `N/A` then the firmware is not enabled. If otherwise (it shows a version for GSP firmware) then the firmware is enabled.

  To disable the GSP firmware, please follow the below steps:

  1. Edit `/etc/default/grub` using `sudo nano /etc/default/grub`
  2. Add `nvidia.NVreg_EnableGpuFirmware=0` inside your `GRUB_CMDLINE_LINUX`
  3. Run `sudo update-grub`
  4. Reboot the system

  See https://forums.developer.nvidia.com/t/major-kde-plasma-desktop-frameskip-lag-issues-on-driver-555/293606 for more information on this issue.
   
* You may be missing the `libnvidia-egl-wayland1` package (which is often recommended). Try installing the package using `sudo apt install libnvidia-egl-wayland1` (**Please** don't do this if you installed version 555+ of the Nvidia driver since the driver installer already installs it for you. The PPA repo packages also install it by default.).
* for Google Chrome (and Chromium-based browsers in general), you may need to switch the "Preferred Ozone platform" flag to "Wayland" or "auto". Follow the steps below in order to apply this:
  1. Go to chrome://flags
  2. Search "Preferred Ozone platform"
  3. Set the flag to "Wayland" or "auto"
  4. Restart the browser
* for some Electron apps, you may need to pass the same Ozone platform flag as we did above. For example `code --enable-features=UseOzonePlatform,WaylandWindowDecorations --ozone-platform-hint=auto` for Visual Studio Code

* You may not have the preserve video memory allocations module parameter enabled, and this can cause issues particularly when suspending and resuming the system, usually in the form of graphical artifacts or a broken desktop environment.

  You can check whether the module parameter is enabled or not by typing `sudo cat /proc/driver/nvidia/params | grep "PreserveVideoMemoryAllocations"`. If the value is `0` or missing, then the parameter is not enabled.

  To enable the preserve video memory allocations module paramter, please follow the below steps:
  
  1. Edit `/etc/default/grub` using `sudo nano /etc/default/grub`
  2. Add `nvidia.NVreg_PreserveVideoMemoryAllocations=1` inside your `GRUB_CMDLINE_LINUX`
  3. Run `sudo update-grub`
  4. Reboot the system
  5. Run `sudo cat /proc/driver/nvidia/params | grep "PreserveVideoMemoryAllocations"` to verify the parameter is now set

  If you are still experiencing issues with suspend/resume after enabling this module parameter, you may want to take a look at Nvidia's [power management documentation](https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/powermanagement.html) to double check that the relevant `systemd` services are installed and enabled.
      
-----

## References
- https://askubuntu.com/questions/206283/how-can-i-uninstall-a-nvidia-driver-completely/206289#206289
- https://askubuntu.com/questions/1391245/getting-the-latest-nvidia-graphics-driver-through-software-updates/1391250#1391250
- https://askubuntu.com/questions/271613/am-i-using-the-nouveau-driver-or-the-proprietary-nvidia-driver
- https://github.com/lutris/docs/blob/master/InstallingDrivers.md
- https://askubuntu.com/questions/66328/how-do-i-install-the-latest-nvidia-drivers-from-the-run-file/66335#66335
- https://askubuntu.com/questions/219942/how-to-uninstall-manually-installed-nvidia-drivers/220729#220729
- https://askubuntu.com/questions/1403854/cant-use-wayland-with-nvidia-510-drivers-on-ubuntu-22-04-lts
- https://bbs.archlinux.org/viewtopic.php?pid=2133404#p2133404
- https://wiki.archlinux.org/title/NVIDIA
- https://wiki.archlinux.org/title/NVIDIA#DRM_kernel_mode_setting
- https://askubuntu.com/questions/68028/how-do-i-check-if-ubuntu-is-using-my-nvidia-graphics-card/624793#624793
- https://www.reddit.com/r/linux_gaming/comments/17fn30q/comment/k6bug3m/
- https://www.reddit.com/r/linux_gaming/comments/17ubgrl/nvidia_libnvidiaeglwayland1_do_i_need_to_install/
- https://www.reddit.com/r/Fedora/comments/rkzp78/make_chrome_run_on_wayland_permanently/
- https://www.reddit.com/r/Fedora/comments/1afkoge/how_to_make_vscode_run_in_wayland_mode/
- https://us.download.nvidia.com/XFree86/Linux-x86_64/555.42.02/README/installationandconfiguration.html
- https://www.reddit.com/r/archlinux/comments/1cxc36m/comment/l528uff/
- https://forums.developer.nvidia.com/t/major-kde-plasma-desktop-frameskip-lag-issues-on-driver-555/293606
- https://download.nvidia.com/XFree86/Linux-x86_64/510.39.01/README/gsp.html
- https://download.nvidia.com/XFree86/Linux-x86_64/435.17/README/powermanagement.html
- https://www.reddit.com/r/linux_gaming/comments/1bjhx8w/explicit_sync_protocol_just_merged_on_wayland/
- https://www.reddit.com/r/linux_gaming/comments/1c9izpc/gnome_461_released_with_explicit_sync/
- https://www.reddit.com/r/linux_gaming/comments/1cx8739/nvidia_555_driver_now_out_explicit_sync_support/
- https://www.tecmint.com/install-nvidia-drivers-in-linux/
- https://www.if-not-true-then-false.com/2015/fedora-nvidia-guide/
- https://www.if-not-true-then-false.com/2015/fedora-nvidia-guide/3/
