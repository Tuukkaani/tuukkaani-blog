+++
title = "Installing Void"
date = "2025-10-03"
categories = ["linux"]
+++

---


### Starting out

Today I wanted to try out void linux. I grabbed the musl live image from voidlinux.org and booted up a vm.
Couple of seconds later "welcome to the void linux live system" popped:

![Welcome](/tuukkaani-blog/img/linux/installing-void/welcome.png)

I decide to log in as root with the credentials `root:voidlinux`. After which I start the `void-installer` then I do the following:

- Change the keyboard to be Finnish by selecting the *fi* from the menu.
- Select the network interface and use DHCP.
- Use the local source for the installation.
- Select the Finnish (tier 1 prkl) XBPS mirror.
- Set the hostname to 'void'.
- Set the timezone to Europe/Helsinki.
- Set the root password.
- Create a user account and leave the groups as default.
- Set the graphical bootloader
- Create the needed partitions using fdisk
  - `g` to create a gpt partitioning table 
  - `n` to create a new partition, partition 1, first sector as default, last sector as `+512M`, `t` to select the type `1`.
  - n to create the second partition, partition 2, first sector as default, last sector as default, type as default.
  - w to write the changes.
- Setup the filesystem
  - Mount the 512M partition as /boot/efi, format as FAT
  - Mount the root partititon as /, format as EXT4

After which I choose the default services to start.

---

### First boot

#### Updating the system

Rebooting I get the tty login. I login as my user account and start by updating the system packages with `sudo xbps-install -Su`

I get the following error:

![xbps update](/tuukkaani-blog/img/linux/installing-void/xbps-error.png)

Luckily I get some easy to follow instructions! I do as I am told and type: 

`sudo xbps-install -u xbps` 

After updating I retry:

`sudo xbps-install -Su`.


#### Installing dbus and elogind

To get these up and running they need to be installed first:

`sudo xbps-install dbus elogind`

Then they need to enabled:

`sudo ln -s /etc/sv/dbus /var/service/`

`sudo ln -s /etc/sv/elogind /var/service/`


#### Installing ufw

Next I'm going to install and configure *ufw* as the firewall. It can be installed with: 

`sudo xbps-install ufw` 

As it finishes installing I set it up:

`sudo sudo ln -s /etc/sv/ufw /var/service/`

`sudo ufw default deny incoming` 

`sudo ufw default allow outgoing`

`sudo ufw reload`

#### For GPUs

Now I would install any GPU driver related packages. The necessary info can be found on [(Voidlinux.org.)](https://docs.voidlinux.org/config/graphical-session/graphics-drivers/index.html).

For my use case I would need drivers for AMD GPU so those would be as follows:

- linux-firmware-amd
- mesa-dri
- vulkan-loader
- mesa-vulkan-radeon
- xf86-video-amdgpu
- mesa-vaapi
- mesa-vdpau

This is everything I would do before installing things like a window manager and so on. Now is a good time to give the system a restart.

---

### Installing DWL 

I've decided to use *dwl* as my window manager for this setup. There are dependencies and tools needed before continuing so 
time to take care of those first.

#### Dependencies

Time to check the dependencies from the dwl repository. 

To install all the base dependencies on void:

`sudo xbps-install base-devel libinput libinput-devel wayland wlroots0.19 wlroots0.19-devel libxkbcommon libxkbcommon-devel wayland-protocols pkg-config`

For xWayland there are few more that are needed:

`sudo xbps-install libxcb libxcb-devel xcb-util-wm xcb-util-wm-devel xorg-server-wayland`

[(Codeberg Dwl.)](https://codeberg.org/dwl/dwl)

Now I'm going to need couple of programs to use dwl properly and to help with the configuration:

`sudo xbps-install foot wmenu vim firefox git font-firacode`

Great! Next it's time to clone the dwl repository:

`git clone https://codeberg.org/dwl/dwl.git`

I need a place to keep all of the configuration / programs:

`mkdir -p ~/.config/suckless && mv dwl ~/.config/suckless/`

#### The first configuration

Ok, time to head to the newly created dwl directory and start configuring.
*config.def.h* is used to configure dwl. After configuration the dwl will be compiled. All of the configuration is done,
on compile time, not during runtime. So once its setup, there is no need to worry about... Right? Lets begin.

First, lets enable xwayland in *config.mk* by uncommenting these two lines:

`XWAYLAND = -DXWAYLAND`

`XLIBS = xcb xcb-icccm`

Next up is the *config.def.h*, here I want to setup the basic keybindings and some functionality that I like.
First in the keyboard section, I add a layout:

`.layout = "fi",`

![Keyboard configuration](/tuukkaani-blog/img/linux/installing-void/keyboard.png)

Few lines below one can find `repeat_rate` and `repeat_delay`. Personally I use `repeat_rate = 50` and `repeat_delay = 200`.

Mouse acceleration is the bane of my existence, but luckily it can be easily turned off by changing:

`LIBINPUT_CONFIG_ACCEL_PROFILE_ADAPTIVE`

to

`LIBINPUT_CONFIG_ACCEL_PROFILE_FLAT`

The default MODKEY is alt, which I do not like so I change the:

`#define MODKEY WLR_MODIFIER_ALT` to `WLR_MODIFIER_LOGO`

Time to configure those keybindings! They can be found in the `Key keys[]` array, I like to do the following:

- MODKEY+q -> Open terminal
- MODKEY+r -> Open wmenu
- MODKEY+c -> Close focused window
- MODKEY+f -> fullscreen

After changing these around it is good to double check for any duplicate bindings!

Finally, time to write the changes to the file and quit out of vim.

#### Final stretch

This is all I need for the bare minimum. It is not everyday useable yet, but it is starting to have the functionality necessary
to start doing things inside the wm, like searches etc.

Time to compile! In the dwl folder use: 

`sudo make clean install`

And it seems everything worked out without errors! To test if it runs I can just type `dwl`.
If everything works like it should, there should be a blank grey screen. 

And voila!

![Fastfetch in foot](/tuukkaani-blog/img/linux/installing-void/fastfetch.png)

---

### Patching DWL & Additional configuration

I know, I know, how can anyone improve on perfection? Lets see. First, I might want to try out a wmenu alternative called mew. Then for a quick patch, I'd like get the slstatus bar going and do some font adjustments to get everything nice and consistent. Maybe change some colors as well.

#### Changing from wmenu to mew

So first, mew. There is one dependency that I need to check:

`sudo xbps-install fcft fcft-devel`

Now I just need to copy the repository and build the thing. 

`git clone https://codeberg.org/sewn/mew.git`

Then a home for the program

`mv mew ~/.config/suckless/`

Aaand

`sudo make clean install`

Now lets change the dwl config to use mew. Quit out of dwl with MODKEY+Shift+q and open up the config.def.h.
In the command section, replace the `"wmenu-run"` with `"mew-run"`.

While at it, lets change the font as well. Mew uses `-f` flag for font.

![Mew font](/tuukkaani-blog/img/linux/installing-void/mew-font.png)

Now throw out the old config.h with `rm` and `sudo make clean install`.

#### Bar.patch

The patch I am going to be using can be found [here](https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/bar).
Right click on the newest release and save link as. I like to save the patches to *dwl/patches*.

All the required dependencies seem to be already installed. Lucky me.

[(Codeberg dwl-patches.)](https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/bar)

Now that we have the *bar.patch* file. Lets navigate to dwl directory and run `patch -i patches/bar.patch`

![Patch error](/tuukkaani-blog/img/linux/installing-void/patch-error.png)

Time to open up the *config.def.h.rej.* file and see what went wrong.

Apparently a keybind failed. I'll just add it manually by *yanking* the line to the *config.def.h*.

While at it, lets give the `*fonts[]` a value of `Fira Code:style=Bold:size=16`.

With that added, I can `rm` the old config.h, .rej and .orig files, then do `sudo make clean install`.

#### Slstatus

Lets get slstatus working. Clone the repository into the suckless directory:

`git clone https://git.suckless.org/slstatus`

Then in the slstatus directory lets run `sudo make clean install`

Now to start dwl with slstatus I need to start dwl again by `slstatus -s | dwl`.

#### Foot.ini

Lets quickly change that font on foot terminal. I need to first create a ~/.config/foot directory.
This is where the *foot.ini* config file will be staying.

`mkdir -p ~/.config/foot`

Then lets copy the premade config file:

`cp /etc/xdg/foot/foot.ini ~/.config/foot/`

In the foot.ini we have a familiar looking font tag. Lets change it to be the same as the rest.

`font=Fira Code:style=Bold:size=16`

There we go, all of the fonts are now nice and consistent!

---

### Sources

Voidlinux.org. Graphics Drivers. Accessed: 10.03.2025. Accessible: https://docs.voidlinux.org/config/graphical-session/graphics-drivers/index.html.
Codeberg Dwl. dwl - dwm for Wayland. Accessed: 10.03.2025. Accessible: https://codeberg.org/dwl/dwl.
Codeberg Mew. mew. Accessed: 10.03.2025. Accessible: https://codeberg.org/sewn/mew.
Codeberg dwl-patches. bar. Accessed: 10.03.2025. Accessible: https://codeberg.org/dwl/dwl-patches/src/branch/main/patches/bar.
