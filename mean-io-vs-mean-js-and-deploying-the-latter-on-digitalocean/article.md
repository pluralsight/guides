## [![meanIOvsJS](assets/meanIOvsJS.jpg)](assets/meanIOvsJS.jpg)

## What are they?

MEAN.(**io** + **js**) are both full-stack JavaScript frameworks, and they are both comprised of these technologies:

* **M**ongoDB – leading NoSQL database, empowering businesses to be more agile and scalable
* **E**xpress – minimal and flexible Node.js web application framework, providing a robust set of features for building single and multi-page, and hybrid web applications
* **A**ngular – lets you extend HTML vocabulary for your application. The resulting environment is extraordinarily expressive, readable, and quick to develop
* **N**ode.js – a platform built on Chrome’s JavaScript runtime for easily building fast, scalable network applications

## Which is which?

There seems to be a lot of confusion between the two, and this [StackOverflow answer](http://stackoverflow.com/questions/23199392/difference-between-mean-js-and-mean-io) can help clarify it. Essentially:

> Mean.js is a fork of Mean.io and both initiatives were started by [the same guy](https://github.com/amoshaviv). Mean.io is now under the umbrella of the company Linnovate and looks like the guy (Amos Haviv) stopped his collaboration with this company and started Mean.js. You can read more about his reasons [here](http://blog.meanjs.org/post/76726660228/forking-out-of-an-open-source-conflict).

On their GitHub pages the MEAN.io has more stars, forks, and watchers but that may easily be to the fact that the MEAN.js is younger:

[![mean_1-1024x892](assets/mean_1-1024x892-790x688.jpg)](assets/mean_1-1024x892.jpg)
MEAN.js GitHub page

[![mean_2-1024x927](assets/mean_2-1024x927-790x715.jpg)](assets/mean_2-1024x927.jpg)
MEAN.io GitHub page

## How to get started?

In this blog post I’ll show you how easy it is to deploy MEAN.js on [DigitalOcean](https://www.digitalocean.com/?refcode=974c9bc93d77). Surely, you can deploy MEAN.io too on [DigitalOcean](https://www.digitalocean.com/?refcode=974c9bc93d77), but you would have to install all the prerequisites on your own, and as you’ll see the MEAN.js is all set up _automagically_ with few button clicks.

**Disclamer:** I am affiliated with [DigitalOcean](https://www.digitalocean.com/?refcode=974c9bc93d77). If the post helped you, you can [signup via my referral link](https://www.digitalocean.com/?refcode=974c9bc93d77). They regularly offer free credit (currently they offer 10$) so you can try it out without paying anything (I’m sure you have the Google skills to get the discount code).

So, from my experience with the cloud hostings I was pretty much amazed how fast you can get your root box up and running on DigitalOcean, additionally they have the (so called) droplet for MEAN ready so in basically less than 1 minute you’ll have everything ready to start coding your MEAN project. Let’s now see how to do this step by step.

## Create a droplet

Naturally, first you have to signup up with [DigitalOcean](https://www.digitalocean.com/?refcode=974c9bc93d77) (remember to look for coupons), and once you do that you’ll be able to login to their web admin dashboard. DigitalOcean calls its virtual servers, **droplets**; each droplet that you create is a new virtual server for your personal use. In order to create a new droplet from your web admin, click on the **CREATE** button, and select a custom hostname and size (as shown on the image below):

[![create_1](assets/create_1-790x532.jpg)](assets/create_1.jpg)

All on the same page (scroll down), select a region which is closest to you (_London  1_ in my case, as shown on the image below):

[![create_2](assets/create_2.jpg)](assets/create_2.jpg)

The most important part is selecting the actual distribution/application that will be installed on your droplet by default. Here you have quite a few options and I encourage you to test them as you see fit. In my example, however, we’re gonna use the **MEAN on Ubuntu 14.04** (as shown on the image below).

[![create_4](assets/create_4.jpg)](assets/create_4.jpg)

Finally, you can leave the additional settings as they are and you can confirm the droplet creation by clicking on the **Create Droplet** button.

[![create_5](assets/create_5.jpg)](assets/create_5.jpg)

## Login to your new droplet

Shortly after (less than **1 minute!**) you’ve clicked the **Create Droplet** button you’ll get the email with the info on how to connect to your droplet. Below is the example of such an email. In order to connect to your droplet you can use [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) (a free implementation of Telnet and SSH for Windows and Unix platforms), and all you’ll have to do is enter the IP and click **Open **(it’s good to name this connection and save it for later use – _digocean_ in the image below).

[![connect_11](assets/connect_11.jpg)](assets/connect_11.jpg)

If the popup appears as shown on the image below, most likely you’ll want to click **Yes**.

[![connect_2](assets/connect_2.jpg)](assets/connect_2.jpg)

When you log in with the username **root** and password that you got in the mail, you’ll be prompted to repeat the root’s password and to change it to something else, which is a standard security practice.

[![connect_3](assets/connect_3.jpg)](assets/connect_3.jpg)

## Run the application

Your application will be located in **/opt/mean** folder, and in order to run it you just have to execute the command 'grunt' in that folder. Output that you’ll most likely see after running the **grunt** command can be seen on the image below:

[![run_1-1024x877](assets/run_1-1024x877-790x676.jpg)](assets/run_1-1024x877.jpg)

To view your application in the browser, visit http://YOUR_DROPLET_IP:3000/ (in my case that would be http://178.62.47.198:3000) and you should see a screen similar to the one on the image below:

[![run_2](assets/run_2-790x568.jpg)](assets/run_2.jpg)

Finally, all done. Next logical step would be to check out the [MEAN.js docs](http://meanjs.org/docs.html).

Happy building!

