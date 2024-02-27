--- 
title: Tool spotlight - Ventoy
date: 2024-02-27 # AANPASSEN
categories: [Tools, Spotlight ] # CATEGORIES always with capitalisation of first letter
tags: [iso-files, open-source, system administration]     # TAG names should always be lowercase
author: CrimsonCloak # Check _data/authors.yml for the correct author names or to add a new author
---

<!-- Add little information for preview snippet -->

Ventoy is an open source tool to create a bootable USB drive that allows for booting from a selection of multiple ISO files. It's a very useful tool for system administrators, IT-enthousiasts and hobbyist that frequently want to boot from different ISO files. It could also be used to install operating systems on the fly without having to constantly reformat and flash your USB drive.


## Pick your device

The only thing you really need in order to use Ventoy, is a designated USB-drive. There are no strict requirements, but it is recommended to use a USB 3.0 drive for faster boot times. Other than that, the only thing to keep in mind is that with more USB-storage, more ISO files can be added to the drive. However, since it is so easy to replace ISO files on your Ventoy, you shouldn't need a very large USB drive (only for bragging rights!).

## Download and install Ventoy

Ventoy is easy to use and can be installed by following the instructions and guide on the offical [GitHub page](https://github.com/ventoy/Ventoy) and [website](https://www.ventoy.net/en/index.html). Roughly the following steps must be performed:

- Download the correct installer and executable for your operating system
- Make sure your USB-device is plugged in and contains no important data
- Run the installer and follow the instructions
- Be sure to select the correct device when installing Ventoy

## Add your ISO files

Using Ventoy is as simple as copying the ISO files to the USB drive. You can easily do this by navigating to your USB device in your file explorer. Here, you can add some of your favorite and/or most commonly used ISO files! This is an example of how it looks on Ubuntu - but it is very similar on most desktop operating systems:

![Ventoy ISO files](/assets/img/ventoy/ventoy_iso.png)



 Simply drop your ISO files in this directory, and it will automatically detect the ISO files and present them in a boot menu when booting from the USB drive!


## Boot from the USB drive into the Ventoy menu

If you have installed Ventoy correctly and added some ISO files, you can now boot from the USB drive. When booting from your Ventoy, you will be presented with a screen similar to this one:

![Ventoy boot menu](/assets/img/ventoy/ventoy_boot_selection.png)

This menu will contain all of the images you have added to your Ventoy. Now all that is left is to select your desired ISO file and boot from it! 


## Conclusion

In short, Ventoy is a nice little tool for people who enjoy tinkering with different operating systems, or for system administrators who need to boot from different ISO files. It is easy to use, and can be installed on any USB drive. As with any open source software, it is free to use and the source code can be viewed on GitHub. If you have tried Ventoy before or are eager to try it, let us know in the comments below!




