Hello my friends.

Virtualbox have give to me a lot of fun and success in my daily work as programmer. Recently I upgraded to Virtualbox 4.2.6 build 82870 in my Windows 8 installation with horrible results. Not only upgrading was a very bad idea : It corrupted the VC++ Redistributables Libraries that other programs uses (I Debug with Nusphere PHPEd my apps in PHP)… If I was able to reinstall PHPEd, Virtualbox was unable to work, and If tried to work with virtualbox, PHPEd refused to debug ignoring the VM… A complete mess! also if I run again the installation of Virtualbox the installer stops almost at the end and after hours of waiting there was no progress and it make me unable even to restart the computer by command (yes, hard shutdown, no recommendable at all).

Having the knife on my neck due to my projects, and few time to act I decided to move to VMWare Workstation… but I had to act very quickly.

I had no time to install, create and configure my new VM or to reinstall my windows, plus I wanted to convert from VDI (The common type of Hard Disk VirtualBox uses by default since 4.1.x), plus the GUI provided for Virtualbox didn’t give me any option to convert the hard disk to VMDK (The default format of VMWare for virtual hard disks).

My only chance was the command line support of VirtualBox, which is excellent and bullet proof. From here we will start exactly 14 steps and around 1 hour and a half if you have a virtual hard disk of 80 GB with support for dynamic disk expansion instead using a big 80GB chunk.

Ready?

##Requirements

- Your virtualbox vm should be off. Detach any network setting.
- Any virtualbox release above of 4.x
- The location of your VDI file in your hard disk.
- Have VMware Workstation 9 or greater installed

##Provided Environment

I’m working in a Windows 8 Pro X64, with 8GB in Ram, 750GB in HD and Intel i7 Core Sandy Bridge Second gen. Mostly all the procedures should work with computers with windows 7 x64, iOS and Ubuntu. There is no difference on the OS, what we need is to Convert the VDI file to VMDK format.

Now lets start with the steps

###Step 1
All this steps must work in any OS, the only difference here is how we reach our installation through the terminal. For this case, I will be executing this under Windows.

Once we located the VDI File with our file manage (called it explorer, or finder, etc) we need to go to the terminal and locate the installation of Virtualbox. In a windows 7/8 X64 machine it should be


    C:\Program Files\Oracle\VirtualBox

so we do

    cd "C:\Program Files\Oracle\VirtualBox" 

once we reach here we need to execute the following line. Remember the location of your VDI file!

    Vboxmanage.exe clonehd "C:\myvm\myharddisk.vdi" "C:\myvm\myharddisk.vmdk" -format VMDK -variant standard 

Once we press enter, the conversion will start. Depending on the size of your hard disk it will take around 1 hour if you have used 50GB of space of your dynamic hard disk expansion in Virtualbox. So you can take your coffee, doing your laundry, or like I did, Play Professor Layton’s Puzzles…

![alt "Command Line"](http://i.imgur.com/BwwrpM2.png "Command Line")

###Step 2

Now that the conversion reached 100%, we can move safely the new vmdk file to a location of our convenience. Run VMware Workstation and start the wizard for creating a new virtual machine.


![alt "Command Line"](http://i.imgur.com/ai9rzYk.png "We should execute a custom installation since we will link the converted VMDK disk in a new machine.")

We should execute a custom installation since we will link the converted VMDK disk in a new machine.

###Step 3
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-2.gif "No changes here, use Workstation 9 as default")

No changes here, use Workstation 9 as default

###Step 4
![alt "Command Line"](http://i1.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-3.gif "We do not need really to reinstall the Operative System, so it is safe to choose “I will Install the operating system later”")

We do not need really to reinstall the Operative System, so it is safe to choose “I will Install the operating system later”

###Step 5
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-4.gif "Choose Linux, then Ubuntu. Sometimes it appears as default")

Choose Linux, then Ubuntu. Sometimes it appears as default

###Step 6
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-5.gif "Name the virtual machine as you want. Also you can change the default location of the virtual machine if needed.")

Name the virtual machine as you want. Also you can change the default location of the virtual machine if needed.

###Step 7
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-6.gif "To match the processors used in my virtualbox virtual machine, I assign the same amount of processors and cores for the new virtual machine")

To match the processors used in my virtualbox virtual machine, I assign the same amount of processors and cores for the new virtual machine

###Step 8
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-7.gif "I do exactly the same like in the previous step. Match the amount of memory to be assign to this virtual machine.")

I do exactly the same like in the previous step. Match the amount of memory to be assign to this virtual machine.

###Step 9
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-8.gif "Use the NAT for the moment. Later on you can configure a bridged network connection to make your VM visible to your network.")

Use the NAT for the moment. Later on you can configure a bridged network connection to make your VM visible to your network.

###Step 10
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-9.gif "Here we will choose the Hard Disk to be used. Locate your converted VMDK disk and continue")

Here we will choose the Hard Disk to be used. Locate your converted VMDK disk and continue


###Step 11
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-10.gif "Give the location of your converted VMDK disk.")

Give the location of your converted VMDK disk.

###Step 12
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-11.gif "Here is the tricky part. Since I don't know if I will use again VirtualBox in a next future, I didn't converted to a recent format. I want to have a Fallback measure if things goes wrong. Once this part is finished, we can test the new virtual machine on VMware.") 

Here is the tricky part. Since I don't know if I will use again VirtualBox in a next future, I didn't converted to a recent format. I want to have a Fallback measure if things goes wrong. Once this part is finished, we can test the new virtual machine on VMware.


###Step 13
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-12.png "Now the only thing we need to do is to start the new Virtual Machine.")

Now the only thing we need to do is to start the new Virtual Machine.

###Step 14
![alt "Command Line"](http://i0.wp.com/www.tbogard.com/wp-content/uploads/2013/01/Page-13.gif "Congratulations! Migration was painless and now I can reinstall PHPEd and go back to work!")

Congratulations! Migration was painless and now I can reinstall PHPEd and go back to work!

Crazy ride isn't? After this last step you need to reconfigure your network and add the VMware additions to your virtual machine.

I hope this tutorial can be helpful if you face migration problems between Virtualbox and VMWare, and I hope you have enjoyed this article.

Thanks!


##About the author

![alt ""](https://secure.gravatar.com/avatar/c633b3f0881e5972f2eaf7b157e95c22.jpg?s=180&d=https%3A%2F%2Fhackhands.s3.amazonaws.com%2Fstatic%2F..%2F_default%2Fassets%2Fimages%2Favatar_large.jpg "")

John Ericson Rodriguez (or Erick if you want to keep it short), is the Founder and CEO of [Stream Overlay Pro](http://www.streamoverlaypro.com). He is an avid software engineer and designer that have worked on many web projects for more than 15 years.

He master the arts of Publishing, Web Developing, GUI Designing, LAMP/WAMP Stacks and knowledgeable in PHP and Javascript frameworks. In his free times, he loves to spend time with his wife and son, ride his bike, creating GUI interfaces experiments with jQueryUI, play e-sports and practice meditation if time allow it. 

[Book a session with me in Hackhands if you have issues that this tutorial does not cover.](http://www.hackhands.com/tbogard)


