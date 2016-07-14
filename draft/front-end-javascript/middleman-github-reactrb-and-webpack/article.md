In this guide we will set up a static webpage using middleman, then host it on github, then give it some interactivity using Reactrb, and finally add some react components using Webpack.

- #### Setting up Middleman
- #### Publishing to Github pages
- #### Write in Reactrb
- #### Using in NPM and Webpack

### Setting up Middleman

1. `gem install middleman`
2. `bundle exec middleman init middleman-fun`
3. `cd middleman-fun`
4. `bundle exec middleman`

Visit localhost:4567 in your browser and you should be rewarded with a nice looking static page.

1. `gem install middleman`

 You will need to have ruby gems installed for this to work.  If you don't middle man has good instructions here:  https://middlemanapp.com/basics/install

 This adds the middleman gem which will give you some new commands, that will initialize new middleman projects, run a server so you can test your changes, and deploy (or publish) your website.

2. `bundle exec middleman init middleman-fun`  

 This will create a new directory named `middleman-fun` inside the current directory.  You can of course change the name to whatever you like.  Then middleman will build a project blank project template.  During the creation process it will ask some questions, just answer yes to all!  *if anybody knows how to default these on the command line, plus do a PR with the answer!*
 
3. `cd middleman-fun`  

 Once we have initialized our project we will want to cd into that directory.

4. `bundle exec middleman`

 This fires up a server, and you will be able to visit your new middleman page.

 Of course you will probably want to use your editor and make some changes just becuase you can.
 
 You will see a directory "source", and in that directory is all the code you will be editing.  Find the file `index.html.erb`, and make some changes.

 Save the file, and you should see the page in your browser automatically reload.

#### Publishing to Github pages

Github pages will serve the gh-pages branch of any repo as a static site.  Perfect for our needs. We will use the [middleman-gh-pages)(https://github.com/edgecase/middleman-gh-pages) gem to automate publishing our middleman site to github pages. 

1. create a github repo
2. add the middleman-gh-pages gem to the projects Gemfile
3. run `bundle install`
3. run `echo "require 'middleman-gh-pages'" >> Rakefile`
4. update the middleman config.rb file
6. run `git add .`
7. run `git commit -am "added middleman-gh-pages"`
8. run `git push origin master`
9. run `bundle exec rake build`
6. run `bundle exec rake publish`

visit *your-github-handle*.github.io/middleman-fun and now your middleman page will be live fo the world to see!

1. creating a github repo

 Before we begin you will need to create a blank github repo.  Lets call it middleman-fun.

 *If you are unfamiliar with github just run through this tutorial as your first step: https://guides.github.com/activities/hello-world/*

 Once you have created your repo, push your local directory to github. *For newbies just follow the instructions github will give you when you create the new repo.*

2. add the middleman-gh-pages gem to the projects Gemfile

 Edit the projects `Gemfile` and add the following line to the bottom of the file: 
 `gem 'middleman-gh-pages'`
 
3. run `bundle install`

 This will fetch the middleman-gh-pages *gem* and install it, and all the other gems it needs.

4. run `echo "require 'middleman-gh-pages'" >> Rakefile`

  This will create a new `Rakefile` and put the single line `require "middleman-gh-pages"` in it.  If you prefer you can do this with your editor of course.
  
  This will effectively create two new *rake* commands which will be used to build and publish our site.

5. update the middleman config.rb file

  Find and edit the `config.rb` file.  At the bottom you will see a section that looks like this:  
```ruby
  configure :build do
    ...
  end
```
  add these two lines *before* the `end`
```ruby
    activate :relative_assets
    set :relative_links, true
```
  This will tell middleman that all your *assets* should have relative links, which github pages requires.

6. run `git add .`
7. run `git commit -am "added middleman-gh-pages"`
8. run `git push origin master`

  These steps will *add* the new Rakefile to your git repo; commit all the changes to the repo; and push the changes to github.
  
9. run `bundle exec rake build`

  This *builds* your static site so that its ready for publication.

10. run `bundle exec rake publish`

  This *pushes* your built site the *gh-pages* *branch* of your github repo.  Github will treat the contents of the branch named *gh-pages* as a static site, and will display it when you visit *your-github-handle*.github.io/middleman-fun
  
At this point you can update your site by editing files in the `source` directory.  You can run `bundle exec middleman` to see your site as you are making changes.  Once you are satisfied and want to publish you will rerun steps (6-10) above.

















### dummy section down here 

some dummy info 






