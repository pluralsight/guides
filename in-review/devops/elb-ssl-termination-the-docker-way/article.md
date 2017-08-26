If you've been working in AWS for long, you've probably run into the problem of ELB SSL termination and how to direct incoming HTTP connections to HTTPS properly.

With a traditional application, this is easy to do with an Nginx or Apache config, but what if you want to do the same in Docker? You could have the logic in your web server container, but what if you don't want the added complexity? The simpler each container is the better, right? So you might think to create an Nginx container that contains only the logic necessary to redirect all incoming connections to https (as I did the first time I came to this problem).

### There's an easier way.

I've created a tiny (5.5MB) Docker container which contains only a single Go app that redirects all incoming requests to HTTPS. All you need to do is run this redirect container alongside your web server container on a different port. Then on the ELB point HTTP:80 at the redirect container, and HTTPS:443 at your own web server container.

//*TODO: 
Explain what was involved?
How would someone else make a similar one with perl/erlang/pyhton app?

```shell
docker run -d -p 8080:80 scottmiller171/go-ssl-redirect:1.1
docker run -d -p 80:80 myRepo/mycontainter
```

I'm looking to make some optimizations to the way the Go app works, and contributions are welcome, but it works wonderfully in my testing.

[GitHub project](https://github.com/smiller171/go-redirect)  
[Docker Hub](https://hub.docker.com/r/scottmiller171/go-ssl-redirect/)  
[![Circle CI](https://circleci.com/gh/smiller171/go-redirect/tree/master.svg?style=svg)](https://circleci.com/gh/smiller171/go-redirect/tree/master)  
[![](https://badge.imagelayers.io/scottmiller171/go-ssl-redirect:latest.svg)](https://imagelayers.io/?images=scottmiller171/go-ssl-redirect:latest 'Get your own badge on imagelayers.io')
