# Arch on Lenovo W520
This is mostly a personal walkthrough to have Arch Linux on Lenovo W520 fully operational as quickly as possible. It might help you in some tough spot, though. This guide will be turned into a script that does all the magic by itself, but I really don't know when. The post-installation issues described below are totally device-specific, and the way I fixed some of the more critical ones was closer to trial and error than to proper investigation and comprehension, so take this with a grain of salt.

#### Installation
Follow the steps described in [beginners' installation guide](https://wiki.archlinux.org/index.php/Beginners'_guide). Nothing special here.

####Post-installation extras
After the system has been installed, there is still a ton of what to do. Some of the following packages can be omitted or replaced with an analog that you like better. These steps should make your interaction with the system more pleasant (but make sure you understand what you're doing). 

* Install ***yaourt***. It will help you find and install packages from Arch User Repository.
First, install  *git*:
```
$ pacman -S git
```
Then install *package-query* and *yaourt* packages:
```
$ git clone https://aur.archlinux.org/package-query.git
$ cd package-query
$ makepkg -si
$ cd ..
$ git clone https://aur.archlinux.org/yaourt.git
$ cd yaourt
$ makepkg -si
$ cd ..
$ rm -rf package-query/ yaourt/
```
To get rid of pkgbuild questions and automate the installation, you can insert this into ~/.yaourtrc:
```
NOCONFIRM=1
BUILD_NOCONFIRM=1
EDITFILES=0
```
* Install ***Xorg*** bundle (initially you'lle need only ***xorg-server*** and ***xorg-xinit***, but the whole bundle will do, too). You may start populating your ***/etc/X11/xorg.conf.d/*** directory with *.conf* files configuring the behaviour of specific devices.

* Install your preferred video drivers and synchronize them with your X configs (cannot elaborate on this yet).

* Install ***xf86-input-synaptics*** to configure touchpad and trackpoint.

* Install ***zsh*** and ***powerline***, then switch shell to zsh with  ```$ chsh -s /bin/zsh```. Copy the example config to ***~/.zshrc***, choose a theme and edit the config in your directory; to refresh the shell with the config, use ```source ~/.zshrc``` command.

* Install your favourite DE or WM. I'm currently using GNOME.
	* Install ***gnome*** bundle from pacman/yaourt/etc.
	* Tune it with a theme like Arc (and enable user themes)
	* Install ***gnome-tweak-tool***
	* Install ***pixel-saver***
	* Install ***dash-to-dock***
	* Many more extensions are available, I'll write down more if I remember.

* Install ***terminator*** --- a terminal emulator with tabs, tiling split screen inside those tabs, customizable themes and layouts.

* Install ***openvpn*** to control VPN.

* Install ***vidalia*** --- a Tor control panel that doesn't require configuration (plug-and-play).
	The installation will fail with a key problem. If you trust the application, you can manually add its key:
```
$ pacman-key -r keyid
$ pacman-key -f keyid
$ pacman-key --lsign-key keyid
```
	Restart the installation with the key verified, it should run smoothly from there on.

* Install ***qbittorrent*** if you are going to use torrents.
	If your traffic is being cut by your provider, do the following:
	* In Options, go to Connections ---> Proxy.
	* Choose type SOCKS5, host 127.0.0.1 and port 9050 (the number must be 1 less than the port in vidalia options).
	* Enable 'Use proxy for peer connections'.
	* Restart qbittorrent to ensure the changes take place.

* Install your favourite browser (you'll probably need a flash plugin from the repos, too).

* Install media applications:
	* ***vlc*** for video and audio
	* ***nomacs*** for photos
	* ***evince*** for book-like documents

####Known Issues
*  ***X window system*** throws errors. If you have both Intel and Nvidia drivers installed and are absolutely sure your X configs are right, you might have the video device wrong in your BIOS. 
	* Enter BIOS --> Display --> Video  Device. Leave the automatic detection of Nvidia Optimus disabled.
 	*   If you want to use integrated Intel graphics, choose *Integrated*.
	* If you prefer the Nvidia card, choose *Discrete*.
	* Better not choose *Nvidia Optimus*, for it will cause you a lot of trouble with X system.

* Arch hangs on load while checking ***/dev/sdX*** or at ***udev*** events, leaving a blinking or frozen cursor and no further indications **or** it hangs when you plug/unplug the adapter **or** when you try to change brightness **or** whatever the hang reason . This means you'll have to start Arch with extra kernel parameters. The inner system conflicts that cannot be completely covered in this guide (I haven't managed to comprehend them yet) cause fatal errors which can fortunately be fixed.
 *Kernel parameters* can be added in many different ways. Your *bootloader* (e.g. GRUB) lets you edit the configuration by pressing a button. You can edit ***/boot/grub/grub.cfg*** manually and insert your parameters there (this will make those changes permanent between reboots). More ways can be found [on Arch wiki](https://wiki.archlinux.org/index.php/Kernel_parameters). Add the parameters as a space-separated list. 
	* ***nox2apic***  --- the lifesaver. Disables the x2APIC controller, which presumably ruins the load when running on discrete video. Lets you break through the initial hangs and get to the login prompt.
	* ***noacpi*** --- disables usage of the ACPI interface in some cases. This can come in handy when you have brightness or power control issues (more information can be found in this [thread](https://bugs.launchpad.net/ubuntu/+source/linux/+bug/776999). 
	* ***nomodeset*** --- instructs the kernel to not load video drivers and use BIOS modes instead until X is loaded. Do not use without a good reason.
	* If you haven't found a suitable parameter here, you can search for it in [this list](http://redsymbol.net/linux-kernel-boot-parameters/3.14/) (substitute the version with your kernel version if necessary).

* When running on integrated video, Intel video drivers prevent X from loading properly.
	* Add the *intel_agp* and *i915* modules to the MODULES line in ***/etc/mkinitcpio.conf***.
	* Regenerate the initramfs with ```mkinitcpio -p linux```.

* Brightness control is disabled after you've managed to stabilize on discrete video. This means that the video card is trying to handle brightness control but is doing it incorrectly. A possible fix (works for me) is to add to your ***/etc/X11/xorg.conf.d/XX-nvidia.conf*** the following line: ```Option "RegistryDwords" "EnableBrightnessControl=1" ```

####To be continued
More useful packages will be specified later, as well as sweet commands, aliases and other hints.