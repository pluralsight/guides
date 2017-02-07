In this guide, we're going to review three services to expose your local Node.js app to the world:

- [localtunnel.me](https://localtunnel.github.io/www/)
- [ngrok](https://ngrok.com/)
- [Zeit's now](https://zeit.co/now)

This can be useful in the following cases:

- When a remote service needs to call your application and you don't want (or can) deploy your application to a public server.
- For debugging purposes, it's better to keep everything local.
- When you're testing mobile apps and you want to expose your local server as the back-end.
- When you want to make a quick demo of your app to a client/user in another part of the world.
- When you want to publicly expose something, like an email server or a file repository from your private network.

Having said that, we'll take a quick look at each option using a Hello World Express app.

The `index.js` file:
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => res.send('Hello World!') );

app.listen(3000, () => console.log('Server listening on port 3000!') );
```

The `package.json` file:
```javascript
{
  "name": "hello",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "4.14.0"
  }
}
```


If you want to skip this and go directly to the comparison table, [click here](#comparison-table).

# localtunnel.me
This is the easiest service you can use to expose your local app to the world. In the [words of the author](https://news.ycombinator.com/item?id=7585056), it was written to easily create a tunnel with no setup, as a library first, and a [CLI](https://en.wikipedia.org/wiki/Command-line_interface) tool second.

Don't confuse it with the [original localtunnel](https://github.com/progrium/localtunnel) that apparently, was discontinued in favor of ngrok.

localtunnel is available as an npm package, so you can install it with:
```
npm install -g localtunnel
```

From any directory, execute the `lt` command:
```
lt --port 3000
```

This will setup a tunnel to you local server on port 3000, and give you a public URL that will remain active even if you restart or stop your server:

![localtunnel start](https://raw.githubusercontent.com/pluralsight/guides/master/images/7c43dd60-ea99-44f3-9ec6-d20fb7a7caae.png)


By default, the URL will have the domain `localtunnel.me` with a subdomain composed of random characters. This URL will change every time you rerun the process.

However, you can request a subdomain (assuming it's available) with the option `--subdomain`. For example:
```
lt --port 3000 --subdomain hello
```

![localtunnel subdomain](https://raw.githubusercontent.com/pluralsight/guides/master/images/7cf21f27-6a79-412e-80e4-c58583de5f6f.png)



In the browser:

![localtunnel browser url](https://raw.githubusercontent.com/pluralsight/guides/master/images/892246e6-e3ed-438d-bb07-199340cf9ebf.png)


As said before, localtunnel can be used as a library also. For example:
```javascript
const localtunnel = require('localtunnel');

const tunnel = localtunnel(port, { subdomain: 'hello'} (err, tunnel) => {
    ...
});

tunnel.on('close', function() {
    // When the tunnel is closed
});

...

tunnel.close();
```

# ngrok
[Ngrok](https://ngrok.com) is the most popular service of all these. Like localtunnel, ngrok creates a secure tunnel to a local server on your machine.

Ngrok is built in Go so it has [binary packages for each major platform](https://ngrok.com/download). You just have to download a ZIP file, unzip it, and then run it from the command line.

That's the recommended way to install it, however, there's also a [Node.js wrapper for ngrok](https://github.com/bubenshchykov/ngrok) that downloads the ngrok binaries for your platform.

To execute ngrok from the directory you unzipped it, run the command:
```
./ngrok http 3000
```

It will start a secure tunnel to your local machine on port 3000 and display a UI in your terminal with the public URL and other information:

![ngrok start](https://raw.githubusercontent.com/pluralsight/guides/master/images/6fc0bf75-c769-41d9-acd5-cf5cf218c34e.gif)


In the browser:

![ngrok browser url](https://raw.githubusercontent.com/pluralsight/guides/master/images/2f4e3864-51eb-4fe7-a2f5-afcf4b273eb1.png)


Ngrok also provides a UI for inspecting *all* the HTTP traffic of your tunnel. Just open a browser and go to http://localhost:4040:

![ngrok ui](https://raw.githubusercontent.com/pluralsight/guides/master/images/b5996792-14e0-4262-9ed6-980dbd054442.png)


This UI also allows you to replay any request, which is very helpful for testing [webhooks](https://webhooks.pbworks.com/w/page/13385124/FrontPage):

![ngrok replay](https://raw.githubusercontent.com/pluralsight/guides/master/images/7b6daf14-27f4-4692-a70a-776371325364.gif)


Just like localtunnel, you get a random subdomain every time you execute ngrok. You can specify a custom subdomain with the `-subdomain` option. For example this command will give you the URL(s) `http(s)://hello.ngrok.io`:
```
./ngrok http -subdomain=hello 3000
```

However, this is only available as a [paying customer](https://ngrok.com/product). It also supports [custom domains](https://ngrok.com/docs#custom-domains).

Among other things, you can configure all the ngrok options of the command line with a YAML file, and it also has a REST API to manage tunnels, collect metrics, and replay requests. You can know more about this features at the [documentation page](https://ngrok.com/docs), which is very complete.

# now

According to its website, [Now](https://zeit.co/now) allows you to deploy your Node.js or Docker-powered app to the cloud with ease, speed, and reliability.

This can give us a hint at the fact that Now uses a different approach than the previous services. Instead of creating a secure tunnel to your localhost, Now deploys you application to the cloud.

You might think this belongs to another category, but the way you can make your application available online without using something like Git, and some features that are helpful for development, make Now enter this comparison.

To install it, we have [three options](https://zeit.co/download#):

- Using pre-built binaries for 64-bit systems.
- Using NPM, you can install Now (globally) with `npm install -g now`
- Using the [Now Desktop app](https://zeit.co/desktop#) (only available for Mac), which also installs the Now CLI.

Once installed, go to your app directory and type `now`. 

The first time you run this command, it will prompt for an email address to create an account and confirm it. Next, it will start deploying your app, creating a URL, and putting it on your clipboard.

It may take a while to deploy your app, but meanwhile, you can go to a browser and paste the URL on your clipboard to see the progress. When it finishes, it will execute the `npm start` script and show your app:

![now app start](https://raw.githubusercontent.com/pluralsight/guides/master/images/a39c9d54-ff4c-4bdd-ba78-e084382f681e.gif)



If we add `/_src` to the URL, we'll see the source code of the deployed application:

![now _src](https://raw.githubusercontent.com/pluralsight/guides/master/images/260c68aa-9916-42f3-8802-5dae92b6628c.gif)


If we make a change to the application, for example:
```javascript
...

app.get('/', (req, res) => res.send('Hello World! v2') );

...
```

And then deploy it with the command `now`, we'll get a new URL where we can see the change. The cool thing is that the previous version will still be available:

![now new and previous version](https://raw.githubusercontent.com/pluralsight/guides/master/images/56c5fe77-4650-41e1-b813-0c7e2c7d4b8e.gif)


The Now CLI provides some other helpful commands. You can see everything that is available with:
```
now -help
```

For instance, there's a `ls` command, to list all the deployed versions of your app:

![now ls](https://raw.githubusercontent.com/pluralsight/guides/master/images/de924654-92dc-4691-822b-22805315d133.png)


You can delete a version with `rm`:

![now rm](https://raw.githubusercontent.com/pluralsight/guides/master/images/ea50e32e-4fac-43e8-9ca5-626901798b46.gif)


Now also supports custom domains and subdomains and it has a [REST API](https://zeit.co/api) to manage all the deployments under the account, domains, certs, aliases, and secrets for the authenticating user.

# Comparison Table

For a quick comparison, here's a summary of the features of each service:

| Feature | localtunnel.me | ngrok | now |
|------------------------|--------------|--------------|--------------------|
| **Free option** | Yes | Yes ([with some limitations](https://ngrok.com/product)) | Yes ([with some limitations](https://zeit.co/now#pricing)) |
| **Custom Subdomains** | Yes (free) | Yes (paid) | Yes (paid) |
| **Custom Domains** | No | Yes (paid) | Yes (paid) |
| **Wildcard domains** | No | Yes (paid) | No |
| **Multiple hosting regions** | No | Yes (paid) | Yes (paid) |
| **Supports HTTPS** | Yes | Yes | Yes |
| **API** | Yes | Yes | Yes |
| **Inspect requests** | No | Yes | No |
| **Replay requests** | No | Yes | No |
| **Support non-Node.js apps** | Yes ([Golang](https://github.com/NoahShen/gotunnelme)) | Yes (anything) | Yes (with Docker) |
| **View Source in the Cloud ** | No | No | Yes |
| **Dynamic Scaling** | No | No | Yes |

# Conclusion

Great, but which one should you use?

Well, all of them are great services, but here are some suggestions to help you make a choice:

- If you are just doing a demo or some quick tests, go with localtunnel. You can install it  with NPM and ask for the same subdomain every time. This helpful if you are setting the URL of your app in an external service. Without paying, you don't have to change it every time.
- If you are testing a webhook, the inspect and replay features of [ngrok](https://ngrok.com/) will be a big help. Also, some frameworks, such as the [Microsoft Bot Framework](https://docs.botframework.com/en-us/tools/bot-framework-emulator/), integrate very nicely with ngrok.
- If you are more interested in the hosting aspect, more than any development-related features, use [Now](https://zeit.co/now).

I hope you found this guide interesting. Please feel free to add any other services that you find useful in exposing Node.JS apps by editing this article and opening a pull request. 

Thanks for reading. 