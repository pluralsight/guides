In this guide we will set up a static webpage using middleman, then host it on github, then give it some interactivity using Reactrb, and finally add some react components using Webpack.

- #### Setting up Middleman
- #### Publishing to Github pages
- #### Write in Reactrb
- #### Using in NPM and Webpack

### Setting up Middleman

```bash
gem install middleman
middleman init middleman-fun
cd middleman-fun
middleman
```

Visit localhost:4567 in your browser and you should be rewarded with a nice looking static page.

```bash
gem install middleman
```

You will need to have ruby gems installed for this to work.  If you don't middle man has good instructions here:  https://middlemanapp.com/basics/install

This adds the middleman gem which will give you some new commands, that will initialize new middleman projects, run a server so you can test your changes, and deploy (or publish) your website.

```bash
middleman initi middleman-fun
```

This will run 