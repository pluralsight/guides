# MySql in Docker for development

Gone are the days when a developer had to make a mess of their base OS by installing loads of software, because Docker has come to save the day.

In this article I'm going to talk about using MySql docker container instead of installng the database server in your system. 

Some advantages of using Docker in this scenario - 
 * Does not start or stop with your system by default. You run it only when you need, it frees up your system's memory and also improves your system start up time.
 * Does not make any changes to your base OS.
 * You can upgrade or downgrade MySql any time you want to without worrying whether the version is in your OS package repository or not.
 * Portable. Easily transferrable from one system to another.
 * Cross-platform. Run the same container on OSX or Linux or Windows
 * I can't think of any more at the moment, although the people at Docker has put up some amazing documentation on their [site](https://docs.docker.com/). Feel free to have a look if you want to know more.