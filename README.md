### Chipidea USB device support for QCA953x ###

Changes the platform to use the Chipidea driver instead of the
generic USB host driver which has support for both host and
device modes (selected on boot) for QCA953x based boards.
It was tested on GL.iNet GL-AR300M-Lite with USB DEVICE mode config.

### Hardware Changes ###

In order to use the USB controller in device mode some soldering is required.
You must pullup gpio13 to 2.5V VCC using a 10K Ohm resistor.

<p class="center">
	<img src="/img/gl-ar300m-lite-usb-device-mode-hw-fix.jpg">
</p>

### Software Changes ###

You need to compile your own LEDE image customized with a patch which adds
support for the USB device mode. Only the trunk version is supported. Here is a
quick guide of the commands

Setup the build environment:

	sudo apt-get update
	sudo apt-get install build-essential subversion libncurses5-dev zlib1g-dev gawk gcc-multilib flex git-core gettext
	mkdir lede
	cd lede
	git clone https://github.com/lede-project/source.git
	cd source

Copy the patch file(if he no yet appears in the current trunk) 940-usb-chipidea-QCA953x-platform-support.patch
to source/target/linux/ar71xx/patches-4.4 dir

Configure:

	make menuconfig

select "Target Profile" - "GL.iNet GL-AR300M", go to "Kernel Modules"/"USB Support" and select(as star)

	kmod-usb-chipidea
	kmod-usb-core
	kmod-usb-gadget-*

<p class="center">
	<img src="/img/lede-config.jpg">
</p>

Exit, save and execute

	make -j<number of cores>

#### Ethernet example: ####
	insmod g_ether idVendor=0x04b3 idProduct=0x4010 dev_addr=00:0D:88:7D:A5:D0 host_addr=00:0E:88:7D:A5:D0
	ifconfig usb0 169.254.64.80
Now you can access the router at address 169.254.64.80 (this is link-local 
address so no need to configure the PC).

Also you can edit file /etc/modules.d/52-usb-gadget-eth:
		usb_f_ecm
		g_ether idVendor=0x04b3 idProduct=0x4010 dev_addr=00:0D:6E:41:D0:A5 host_addr=00:0E:6E:41:D0:A5

Strings "idVendor=0x04b3 idProduct=0x4010" are needed to make Microsoft Windows happy :-)

#### Serial example: ####
	insmod g_serial
	echo 'ttyGS0::askfirst:/bin/ash --login' >> /etc/inittab
	kill -HUP 1 #reloads inittab
Connect to the newly detected serial port and press enter in the blank screen.

#### Mass Storage: ####
	dd if=/dev/zero of=/tmp/mass_storage.img bs=1M count=10
	insmod g_mass_storage file=/tmp/mass_storage.img removable=1 cdrom=0
In windows this will pop format dialog since the image is not initialized.

### References ###
Special thanks to
[Svyatoslav Neykov](https://github.com/neykov) for his initial work on AR933x and ChipIdea.
