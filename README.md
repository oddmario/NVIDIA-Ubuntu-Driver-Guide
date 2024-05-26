# NVIDIA Ubuntu Driver Guide
A little guide to help you install the NVIDIA GPU drivers on your Ubuntu system(s)

## Driver installation

### Installing through the `graphics-drivers` PPA repository
1. Ensure that you have uninstalled any previously installed NVIDIA drivers by running the below commands:
```
sudo apt-get remove --purge '^nvidia-.*'
sudo apt autoremove
reboot
```

2. Add the repository and install the driver:
```
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt update 
sudo apt install nvidia-driver-550
sudo reboot
```

NOTE: At the time of writing this guide, 550 is the latest version of the driver. Navigate to https://launchpad.net/~graphics-drivers/+archive/ubuntu/ppa to check what is the latest version of the driver then replace the `driver-550` part with the version you would like to install.

3. Once the system has rebooted, run `nvidia-smi` to confirm that the driver has been installed with no issues.

### Installing through the official NVIDIA installer from the Nvidia.com website
Please do note that this procedure is more advanced and is often not recommended, even though it shall go alright as long as you follow each step with patience and care :)

1. Ensure that you have uninstalled any previously installed NVIDIA drivers by running the below commands:
```
sudo apt-get remove --purge '^nvidia-.*'
sudo apt autoremove
reboot
```

2. Navigate to https://www.nvidia.com/Download/index.aspx?lang=en-us and download the proper driver for your GPU and Linux architecture. The website should give you a file that ends with the `.run` file extension.

**NOTE:** It would be lovely to store the downloaded `.run` file in a permanent place because you will need the exact same file if you would like to uninstall the driver later.

3. Switch to the terminal of your system by pressing `Ctrl + Alt + F3` (if this does not switch from the GUI mode to the terminal mode, try `Ctrl + Alt + F1` or `Ctrl + Alt + F2` instead)

4. Stop the GDM service:
```
sudo systemctl stop gdm
sudo systemctl stop gdm3
```
If this fails for you, try `sudo systemctl stop lightdm` instead.

5. Change to the path of the directory that includes the downloaded `.run` file using `cd`

6. Run the installer:
```
chmod +x NVIDIA-Linux-x86_64-555.42.02.run
sudo sh ./NVIDIA-Linux-x86_64-555.42.02.run
```

7. The installer will guide you through everything.
   
NOTE: If the installer asks you to disable Nouveau, allow the installer to disable it for you. You may need to abort the installer after this, then run `sudo update-initramfs -u && reboot`, then follow steps 3 to 6 above in order to restart the installer again once the system has completed rebooting.

8. Once the installer has completed installing the driver, run `reboot` to reboot your system. Your newly installed driver should be up and running once the system boots up (you may run `nvidia-smi` to confirm so).

-----

## Driver uninstallation

### Uninstalling the driver when installed through the `graphics-drivers` PPA repository
Run:
```
sudo apt-get remove --purge '^nvidia-.*'
sudo apt autoremove
reboot
```

### Uninstalling the driver when installed through the official NVIDIA installer from the Nvidia.com website
1. Switch to the terminal of your system by pressing `Ctrl + Alt + F3` (if this does not switch from the GUI mode to the terminal mode, try `Ctrl + Alt + F1` or `Ctrl + Alt + F2` instead)

2. Stop the GDM service:
```
sudo systemctl stop gdm
sudo systemctl stop gdm3
```
If this fails for you, try `sudo systemctl stop lightdm` instead.

3. Change to the path of the directory that includes the downloaded `.run` file using `cd` (NOTE: Make sure its the exact same `.run` file that you used to install the driver)

4. Run the uninstaller:
```
chmod +x NVIDIA-Linux-x86_64-555.42.02.run
sudo sh ./NVIDIA-Linux-x86_64-555.42.02.run --uninstall
```

-----

## Issues faced after installing the NVIDIA drivers, and how to solve them

- There's a ghost "Unknown Display" on the GNOME Displays settings (especially if you followed the `graphics-drivers` PPA repository installation procedure).
  
  This seems to be a bug reported at https://bugs.launchpad.net/ubuntu/+source/nvidia-graphics-drivers-535/+bug/2063222

  A workaround is:
  ```
  [ Workaround ]
  
  1. sudo rm /dev/dri/card0
  2. Log in again.
  ```

- Wayland is not shown as an option on the login screen (or the cog icon of the login screen doesn't show at all)

  1. Edit the `/etc/gdm3/custom.conf` file using `sudo nano /etc/gdm3/custom.conf`
  2. Ensure that `WaylandEnable=true` is set in that file and make sure that it's uncommented (does not start with a `#`)
  3. Run `sudo ln -s /dev/null /etc/udev/rules.d/61-gdm.rules`
  4. Reboot the system

- The experience on Wayland is not the smoothest

  Your system may be using the Mesa driver instead of the NVIDIA one on Wayland sessions. You can confirm this by typing `glxinfo|egrep "OpenGL vendor|OpenGL renderer*"`

  In order to solve this:
  
  1. Edit `/etc/default/grub` using `sudo nano /etc/default/grub`
  2. Add `nvidia-drm.modeset=1` inside your `GRUB_CMDLINE_LINUX` (i.e. `GRUB_CMDLINE_LINUX="nvidia-drm.modeset=1"`)
  3. Run `sudo update-grub`
  4. Reboot the system
 
-----

## References
- https://askubuntu.com/questions/206283/how-can-i-uninstall-a-nvidia-driver-completely/206289#206289
- https://askubuntu.com/questions/1391245/getting-the-latest-nvidia-graphics-driver-through-software-updates/1391250#1391250
- https://askubuntu.com/questions/271613/am-i-using-the-nouveau-driver-or-the-proprietary-nvidia-driver
- https://github.com/lutris/docs/blob/master/InstallingDrivers.md
- https://askubuntu.com/questions/66328/how-do-i-install-the-latest-nvidia-drivers-from-the-run-file/66335#66335
- https://askubuntu.com/questions/219942/how-to-uninstall-manually-installed-nvidia-drivers/220729#220729
- https://askubuntu.com/questions/1403854/cant-use-wayland-with-nvidia-510-drivers-on-ubuntu-22-04-lts
- https://askubuntu.com/questions/68028/how-do-i-check-if-ubuntu-is-using-my-nvidia-graphics-card/624793#624793
- https://www.reddit.com/r/linux_gaming/comments/17fn30q/comment/k6bug3m/
