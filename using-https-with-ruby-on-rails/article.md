![](http://i.imgur.com/1CLlQld.png)


HTTPS is the secure, encrypted version of the HTTP protocol. To serve a Ruby on Rails application via HTTPS there are 3 steps you need to follow:

1. Obtain an SSL certificate
2. Configure the web server to use the SSL certificate
3. Configure the Ruby on Rails application for HTTPS

In this tutorial we'll take a look at each of these steps. Since there are several possible alternatives to serve a Ruby on Rails application via HTTPS, here we'll focus on the most common options and configurations.

If you are migrating an existing Ruby on Rails application from HTTP to HTTPS, there are a number of small changes and best practices you should follow to simplify the transition and guarantee the compatibility with both protocols. I'll mention a few of them at the end of the tutorial.


## Obtaining an SSL certificate

There are [several different types of SSL certificates](https://simonecarletti.com/blog/2013/11/ssl-certificate-types/). You can group them by validation level (domain validated, organization validated, extended validation), by coverage (single-name, wildcard, multi-domains, etc.) and authenticity (self-signed vs publicly-trusted certificate authorities).

In general, *single-name* or *wildcard* certificates are the most common choices. They allow you to secure a single hostname (such as `www.example.com`) or an entire subdomain level (such as `*.example.com`). Unless you need some extra level of validation, *domain validated certificates* are the cheapest and most common solution. The second most-popular alternative are the *extended validation* certificates, which are generally recognized by the green bar displayed by the browsers in the address bar.

![image](https://d2mxuefqeaa7sj.cloudfront.net/s_5208C718E585F4ACF18509DB8B9E7D16BE8F660F1FA1C856DE8EB55FF12C12C8_1451482192246_Screen_Shot_2015-12-30_at_14_28_25.png)

For production environments, you are required to purchase an SSL certificate issued by a trusted certificate authority (e.g. [Digicert](https://www.digicert.com/), [Comodo](https://www.comodo.com/), [Let's Encrypt](https://letsencrypt.org/)) or a reseller (e.g. [DNSimple](https://dnsimple.com/ssl-certificates)). To purchase a trusted SSL certificate, follow the instructions provided by the certificate provider.

For non-production applications, you can avoid the costs associated with the SSL certificate by using a self-signed SSL certificate. If you use a self-signed certificate the connection will still be encrypted, however your browser will likely display a security warning because the certificate is not issued by a trusted certification authority. You can [follow these instructions](https://devcenter.heroku.com/articles/ssl-certificate-self) to generate a self-signed certificate.

Regardless the type of certificate, at the end of the issuance process you should obtain the following files:

1. The public SSL certificate
2. The private key
3. Optionally, a list of intermediate SSL certificates or an intermediate SSL certificate bundle

These files are required to proceed to the next step and configure the web server to support HTTPS.


## Configuring the web server to support HTTPS

In this section we'll learn how to configure the most common Ruby on Rails web servers to serve an application under HTTPS. We'll use the public SSL certificate, the private key and the intermediate SSL chain we obtained at the previous step.

For the purpose of the examples, I'll use the following file names:

- `certificate.crt` - the public SSL certificate
- `private.key` - the private key

Depending on the web server, you may need to supply the SSL intermediate chain in a single file along with the public SSL certificate, or use two separate files:

- `chain.pem` - the intermediate SSL certificate bundle
- `certificate_and_chain.pem` - the SSL certificate and intermediate SSL certificate bundle


### Intermediate and SSL Certificate Bundle

The creation of the intermediate SSL certificate bundle is generally one of the most confusing step, therefore it deserves a special mention.

The bundle is just a simple text file that contains the concatenation of all the intermediate SSL certificates. The order is generally in reverse order, from the most specific intermediate SSL certificate to the most generic (and/or the root certificate).

The root certificate is generally omitted as it should be bundle in the browser or in the operating system. If the bundle has to contain the server SSL certificate, then this must appear as the first certificate in the list (as this is the most specific).

```
SERVER CERTIFICATE
INTERMEDIATE CERTIFICATE 1
INTERMEDIATE CERTIFICATE 2
INTERMEDIATE CERTIFICATE N
ROOT CERTIFICATE
```

You can use a text editor to concatenate the files together, or the `cat` unix utility.

```
cat certificate.crt interm1.crt intermN.crt root.csr > certificate_and_chain.pem
cat interm1.crt intermN.crt root.csr > chain.pem
```

### Nginx

Nginx is generally used as a front-end proxy for other Rack web servers, such as Unicorn, Passenger, Puma or Thin.

To configure HTTPS on Nginx, the `ssl` parameter must be enabled on listening sockets in the server block, and you must specify the locations of the SSL certificate bundle and private key files:

```
server {
    listen 443 ssl;
    server_name www.example.com;
    ssl_certificate /path/to/certificate_and_chain.pem;
    ssl_certificate_key /path/to/private.key;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ...
}
```

See also http://nginx.org/en/docs/http/configuring_https_servers.html. You may want to refer to [cipherli.st](https://cipherli.st/) for the recommended cipher and protocol configurations.

Although you can configure a server block to listen on both `80` and `443`, it's generally a common practice to configure two different server blocks: one for HTTP and one for HTTPS.

Don't forget to restart the web server to apply the configurations.

### Apache

Apache is generally used in combination with Passenger.

To configure HTTPS on Apache, the `VirtualHost` must listen to the port `443`, you must turn on the `SSLEngine` and specify the locations of the SSL certificate bundle and private key files.

```
<VirtualHost *:443>
    ServerName www.example.com
    SSLEngine on
    SSLCertificateFile "/path/to/certificate_and_chain.pem"
    SSLCertificateKeyFile "/path/to/private.key"
</VirtualHost>
```

If you are using a version of Apache before 2.4.8, please note that [you need to use two separate files for the public SSL certificate and the SSL intermediate chain](https://httpd.apache.org/docs/2.4/mod/mod_ssl.html#sslcertificatechainfile).

```
<VirtualHost *:443>
    ServerName www.example.com
    SSLEngine on
    SSLCertificateFile "/path/to/certificate.crt"
    SSLCertificateChainFile "/path/to/chain.pem"    
    SSLCertificateKeyFile "/path/to/private.key"
</VirtualHost>
```

See also https://httpd.apache.org/docs/2.4/ssl/ssl_howto.html. You may want to refer to [cipherli.st](https://cipherli.st/) for the recommended cipher and protocol configurations.

Don't forget to restart the web server to apply the configurations.

### Heroku

Heroku is a popular platform as a service (PaaS) also used to deploy Rack applications. Heroku is a managed environment, therefore you don't get access to the low-lever web server configuration.

In order to configure the SSL certificate on Heroku you have to use the Heroku CLI to [provision the SSL endpoint](https://devcenter.heroku.com/articles/ssl-endpoint#provision-the-add-on)

```shell
$ heroku addons:create ssl:endpoint
```

and install the SSL certificate using the private key and the full SSL certificate bundle composed by the certificate and the chain (if any).

```shell
$ heroku certs:add certificate_and_chain.pem private.key
```

### Amazon AWS

How you configure AWS to serve HTTPS depends on the specific AWS service you use. If you use a single EC2 instance, then you'll have to install a web server to host your Ruby on Rails application and the configuration depends on the specific server you will use. If you use Nginx or Apache along with a specific Rack web server, you can follow the instructions published in the previous sections.

See also [configuring HTTPS for Single-Instance Environments](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/https-singleinstance.html) and [configuring HTTPS on single instances of Ruby](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/https-singleinstance-ruby.html) .

## Configuring the Ruby on Rails application for HTTPS

By default, once you properly configured your webserver for HTTPS, you should be able to load your Ruby on Rails application via HTTPS without further configurations.

However, to effectively serve a Ruby on Rails application via HTTPS, it is recommended that you flag the cookies as `secure`, respond with the appropriate HTTPs security headers and redirect HTTP traffic to HTTPS.

### `force_ssl` configuration

In the recent Ruby on Rails versions, the framework provides a configuration flag called `force_ssl` you can enable to force the application to run under HTTPS. This feature is available since Ruby on Rails 3.1.

You can turn this feature on in a specific environment by setting the value to `true` in the environment file.

```ruby
# config/environments/production.rb
Rails.application.configure do
  # ...

  # force HTTPS on production
  config.force_ssl = true
end
```

Please remember to restart your Ruby on Rails web server to apply the changes.

You can also turn on the feature globally in the `application.rb` file, and this will apply to all the environments. However, be aware that this will also affect your `test` and `development` environments.

```
module MyApp
  class Application < Rails::Application

  # ...

  # force HTTPS on all environments
  config.force_ssl = true
end
```

Once enabled, the framework will perform the following actions for you:

1. All cookies set by the application [will be flagged as secure](http://tools.ietf.org/html/rfc6265#section-4.1.2.5)
2. The response will contain the [HTTP Strict Transport Security (HSTS) security header](https://en.wikipedia.org/wiki/HTTP_Strict_Transport_Security) that will instruct the browser to perform subsequent requests via HTTPS only
3. All HTTP requests will be redirected to HTTPS

It's important to note that this configuration applies to the entire environment: it affects all request sent to the application and it's not possible to enable/disable it on specific controllers or actions. Furthermore, it's not possible to run the application under HTTP and HTTPS in parallel because of the point (3).

### The `rack-ssl` gem

If you use a Ruby on Rails version lower than 3.1 or if you want a little bit of more control, you can use the [`rack-ssl`](https://rubygems.org/gems/rack-ssl) . This gem is a `Rack` middleware that provides exactly the same feature of the `force_ssl` feature. In fact, Ruby on Rails [used to rely on `rack-ssl` when the `force_ssl` feature was introduced](https://github.com/rails/rails/commit/2c0c4d754e34b13379dfc53121a970c25fab5dae).

To use this gem in a Ruby on Rails application, add the dependency to the `Gemfile`

```ruby
gem 'rack-ssl', require: 'rack/ssl'
```

and install it with `Bundler`

```shell
$ bundle
```

Then, add the middleware to the application middleware stack:

```ruby
# config/application.rb
config.middleware.insert_before ActionDispatch::Cookies, "Rack::SSL"
```

It is recommended that you add the middleware in a position at least before `ActionDispatch::Cookies` and `ActionDispatch::Static` (if loaded), because the middleware may need to optimize the response from those two middleware for HTTPS.

`Rack::SSL` is a little bit more flexible than the `force_ssl` configuration. You can enable it on a per-request basic by passing the `:exclude` option.

The following example disables the middleware if the request comes from HTTP and it allows to run the application in parallel to HTTP and HTTPS.

```ruby
config.middleware.insert_before ActionDispatch::Cookies, "Rack::SSL",
                                exclude: ->(env) { !Rack::Request.new(env).ssl? }
```

This approach is particularly useful to silently rollout HTTPS in parallel to the HTTP version of the site, to test how your application behaves and fix bugs before forcing the entire site to HTTPS.

You can also configure the HSTS header by setting, for example, a different expiration and to include subdomains.

```ruby
config.middleware.insert_before ActionDispatch::Cookies, "Rack::SSL",
                                hsts: { expires: 2.years, subdomains: true }
```

Or you can disable the HSTS header completely. This is useful if the header is already set at another level, for instance by a front-end proxy such as HAProxy or Nginx.


```ruby
config.middleware.insert_before ActionDispatch::Cookies, "Rack::SSL",
                                hsts: false
```

Of course, you can combine these options together.

```ruby
config.middleware.insert_before ActionDispatch::Cookies, "Rack::SSL",
                                exclude: ->(env) { !Rack::Request.new(env).ssl? },
                                hsts: false
```

You can also inject the middleware in a specific environment to enable it only for that specific environment. Alternatively, you can use the `exclude` configuration in combination with an environment check.

Although the `rack-ssl` gem is a little bit more customizable, it still works at Rack middleware level therefore it's a little bit tricky to selectively enable or disable it on a per-controller or per-action basis. You could in theory use the `exclude` option to filter by request path, but this approach may quickly become cumbersome if you have a large number of routes.


### HTTPS configuration per-controller or per-action

There are possible solutions to force HTTPS only on specific actions or controllers. The most common approach is to use a `before_action` to force HTTPS whenever required, however I personally discourage this approach as it drastically reduces the security of your application, it increases the complexity of your controllers and it limits the ability to use extra security measures such as the security headers. Please consider to serve the entire application via HTTPS, not just a portion of it.

In certain cases, a Ruby on Rails application is a container for more than one application or different components. Let's think for a second about Rails applications that consists in a front-end and an API, or a front-end where the assets are served from different domains.

There may be cases where you need to enable HTTPS only on certain actions or controllers that represent one or more specific components. In these cases, it is recommended that you map these components to separate hostnames.

In other words, if the APIs are served via HTTPS and the website via HTTP, but they are both powered by the same application, you should use

```
https://api.example.com/
http://example.com
```

instead of

```
https://example.com/api/
http://example.com
```

as security policies such as the [`HSTS`](https://developer.mozilla.org/en-US/docs/Web/Security/HTTP_strict_transport_security) and the [`HPKP`](https://developer.mozilla.org/en-US/docs/Web/Security/Public_Key_Pinning) [security headers](https://securityheaders.io/) applies to an entire hostname.

It's also quite easy to enable the `rack-ssl` middleware only for a list of hostnames, rather than having to deal with a controllers and actions.

## Tips & Tricks

The following is a list of suggestions and best practices you should follow to reduce the effort of migrating an existing Ruby on Rails applications from HTTP to HTTPS. Some of these best practices applies to Ruby on Rails or web development in general, and you can already follow them today regardless your plans to move the application under HTTPS.

### Use the application router to dynamically generate URLs and links

One of the most important rule to follow to ensure the long-term maintainability of a Ruby on Rails application, aside from HTTPS, is to never hard-code the application routes anywhere in the code. Instead, you should always rely on the router to generate links and URLs.

It's not uncommon to see hard-code links like this one in mailer templates.

```
If you have any questions, feel free to <a href="http://example.com/contact">contact us</a>.
```

Another very common habit is to hard-code application URLs in the JavaScript files or static assets.

It's very important to always rely on the [Ruby on Rails routing helpers](http://guides.rubyonrails.org/routing.html) whenever you need to generate a route. The most essential routing helper is [`url_for`](http://api.rubyonrails.org/classes/ActionDispatch/Routing/UrlFor.html#method-i-url_for) which is very basic, but extremely powerful.

For named routes, Ruby on Rails provides the `_path` and `_url` helper. For example, if you have a route called `posts` that maps to `/posts`, then you can use:

- `posts_path` to generate `/posts`
- `posts_url` to generate `http://example.com/posts`

The `_url` helper is really powerful because it automatically constructs the full absolute link using the current request protocol and hostname. Therefore, if you use these helpers to generate absolute links, Ruby on Rails will automatically display HTTPS links if the request was for made via HTTPS.

This works very well in templates that are generated at runtime by the server. But what about a static file such as a JavaScript file that needs to send a request to the application? I mentioned before that it's not uncommon to see routes hard-coded in JavaScript files or CSS.

For JavaScript files, you can rely on the HTML data attribute to pass information to the script. Let's suppose you have a script that needs to send a POST request when the user clicks a button.

```
// somefile.js
$.post("https://example.com/loader", function(payload) {
  $("#result").html(payload);
});

<!-- file.html --> 
<div id="result"></div>
```

Instead, generate and render the URLs in the template using the `_url` helper

```
<!-- file.html --> 
<div id="result" data-url="<%= loader_url %>"></div>
```

and change the JavaScript to fetch the target URL from the DOM.

```
// somefile.js
$.post($("#result").data("url"), function(payload) {
  $("#result").html(payload);
});

<!-- file.html --> 
<div id="result"></div>
```

### Use relative paths whenever possible


If you use relative paths from the root, the browser will automatically resolve the links by appending the current request hostname and protocol to the path.

For example, replace

```html
<link rel="shortcut icon" type="image/x-icon" href="http://example.com/images/favicon.png" />
```

with

```html
<link rel="shortcut icon" type="image/x-icon" href="/images/favicon.png" />
```

This is not required if you use the Ruby on Rails routing helpers I mentioned in the previous section.

### Load third-party assets via HTTPS whenever possible

The most common error you will experience while transitioning from HTTP to HTTPS is the [mixed content warning](http://www.howtogeek.com/181911/htg-explains-what-exactly-is-a-mixed-content-warning/). This happens when your site is using HTTPS but you are loading an external resource (such as an image or a JavaScript file) via HTTP.

To prevent this issue, always link to the HTTPS version of a third party resource whenever you can. That will save you some headache when you'll switch your website to HTTPS. In fact, in general there is absolutely no downside of embedding an HTTPS resource in a site loaded via HTTP, whereas the vice-versa is not true.

Most content providers are now distributing their resources both via HTTP and HTTPS. For example, let's suppose you want to embed a font via Google Font or jQuery from the `code.jquery.com` CDN: in both cases, you can link immediately to the HTTPS version of the resource, there is no reason to use the HTTP link.

In the past, it was also quite common to use the [protocol-relative link](http://www.paulirish.com/2010/the-protocol-relative-url/) to delegate to the browser the decision to use http or https depending on the request.

```html
<link rel="stylesheet" media="screen" href="//netdna.bootstrapcdn.com/font-awesome/4.3.0/css/font-awesome.css" />
```

However, now that SSL is encouraged for everyone and doesn't have performance concerns, this technique is considered an anti-pattern. If the asset you need is available on SSL, then always use the `https://` link as I just mentioned before.

### Configure the browser security headers

Both `force_ssl` and `rack-ssl` options are designed to quickly setup your application to get started with HTTPS with a default set of options and security configurations.

However, now that your Ruby on Rails application is running via HTTPS, you may want to fully take advantage of HTTPS and properly tweak the security headers returned by your application. On this topic, check out the [introduction to Browser Security Headers](https://www.pluralsight.com/courses/browser-security-headers) from Troy Hunt. This course will help you to better understand what are the security headers related to HTTPS and how to properly use them.

### About the author:

[Simone Carletti](https://simonecarletti.com/) is a passionate programmer and scuba diving instructor. He works at DNSimple, and created [RoboWhois](https://www.robowhois.com/) and [RoboDomain](http://robodomain.com/). Simone have been involved with software development for more than a decade, contributing code and creating libraries. He is [#1 answerer](http://stackoverflow.com/users/123527/simone-carletti) for Rails questions on StackOverFlow.com.
