Hello my friends.

Virtualbox have give to me a lot of fun and success in my daily work as programmer. Recently I upgraded to Virtualbox 4.2.6 build 82870 in my Windows 8 installation with horrible results. Not only upgrading was a very bad idea : It corrupted the VC++ Redistributables Libraries that other programs uses (I Debug with Nusphere PHPEd my apps in PHP)… If I was able to reinstall PHPEd, Virtualbox was unable to work, and If tried to work with virtualbox, PHPEd refused to debug ignoring the VM… A complete mess! also if I run again the installation of Virtualbox the installer stops almost at the end and after hours of waiting there was no progress and it make me unable even to restart the computer by command (yes, hard shutdown, no recommendable at all).

Having the knife on my neck due to my projects, and few time to act I decided to move to VMWare Workstation… but I had to act very quickly.

I had no time to install, create and configure my new VM or to reinstall my windows, plus I wanted to convert from VDI (The common type of Hard Disk VirtualBox uses by default since 4.1.x), plus the GUI provided for Virtualbox didn’t give me any option to convert the hard disk to VMDK (The default format of VMWare for virtual hard disks).

My only chance was the command line support of VirtualBox, which is excellent and bullet proof. From here we will start exactly 14 steps and around 1 hour and a half if you have a virtual hard disk of 80 GB with support for dynamic disk expansion instead using a big 80GB chunk.

Ready?

##Requirements

+Your virtualbox vm should be off. Detach any network setting.
+Any virtualbox release above of 4.x
+The location of your VDI file in your hard disk.
+having installed VMware Workstation 9 or superior

##Provided Environment

I’m working in a Windows 8 Pro X64, with 8GB in Ram, 750GB in HD and Intel i7 Core Sandy Bridge Second gen. Mostly all the procedures should work with computers with windows 7 x64, iOS and Ubuntu. There is no difference on the OS, what we need is to Convert the VDI file to VMDK format.

Now lets start with the steps

## 1. Startup.
All this steps must work in any OS, the only difference here is how we reach our installation through the terminal. For this case, I will be executing this under Windows.

Once we located the VDI File with our file manage (called it explorer, or finder, etc) we need to go to the terminal and locate the installation of Virtualbox. In a windows 7/8 X64 machine it should be


    C:\Program Files\Oracle\VirtualBox

so we do

    cd "C:\Program Files\Oracle\VirtualBox" 

once we reach here we need to execute the following line. Remember the location of your VDI file!

    Vboxmanage.exe clonehd "C:\myvm\myharddisk.vdi" "C:\myvm\myharddisk.vmdk" -format VMDK -variant standard 

Once we press enter, the conversion will start. Depending on the size of your hard disk it will take around 1 hour if you have used 50GB of space of your dynamic hard disk expansion in Virtualbox. So you can take your coffee, doing your laundry, or like I did, Play Professor Layton’s Puzzles…

![alt "Command Line"](http://i.imgur.com/BwwrpM2.png "Command Line")

###Step 1.

Now that the conversion reached 100%, we can move safely the new vmdk file to a location of our convenience. Run VMware Workstation and start the wizard for creating a new virtual machine.

![alt "Command Line"](http://i.imgur.com/ai9rzYk.png "We should execute a custom installation since we will link the converted VMDK disk in a new machine.")

We should execute a custom installation since we will link the converted VMDK disk in a new machine.

