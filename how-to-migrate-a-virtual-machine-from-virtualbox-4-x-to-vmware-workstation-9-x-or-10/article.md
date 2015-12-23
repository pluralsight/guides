Hello my friends.

Virtualbox have give to me a lot of fun and success in my daily work as programmer. Recently I upgraded to Virtualbox 4.2.6 build 82870 in my Windows 8 installation with horrible results. Not only upgrading was a very bad idea : It corrupted the VC++ Redistributables Libraries that other programs uses (I Debug with Nusphere PHPEd my apps in PHP)… If I was able to reinstall PHPEd, Virtualbox was unable to work, and If tried to work with virtualbox, PHPEd refused to debug ignoring the VM… A complete mess! also if I run again the installation of Virtualbox the installer stops almost at the end and after hours of waiting there was no progress and it make me unable even to restart the computer by command (yes, hard shutdown, no recommendable at all).

Having the knife on my neck due to my projects, and few time to act I decided to move to VMWare Workstation… but I had to act very quickly.

I had no time to install, create and configure my new VM or to reinstall my windows, plus I wanted to convert from VDI (The common type of Hard Disk VirtualBox uses by default since 4.1.x), plus the GUI provided for Virtualbox didn’t give me any option to convert the hard disk to VMDK (The default format of VMWare for virtual hard disks).

My only chance was the command line support of VirtualBox, which is excellent and bullet proof. From here we will start exactly 14 steps and around 1 hour and a half if you have a virtual hard disk of 80 GB with support for dynamic disk expansion instead using a big 80GB chunk.

Ready?