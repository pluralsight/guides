![](http://i.imgur.com/4wFa7JX.png?1)

When starting out with Front-end programming one will run across Angular and React pretty quickly. Currently the JavaScript scene is changing at a very rapid pace, and new technology is coming out every day it seems like. While some may think this is a good thing and others a bad thing, that is besides the point. At this point in time it seems that Angular and React are the two big players that everyone is using. In 6 months or a year that could all change, and as a programmer you should be able to adapt to that change easily. 

There are a lot of articles out there about Angular vs React, and a lot of them tend to take a biased side to either one or the other. We can argue that Angular is more popular, and that React makes you a better JavaScript programmer all day long. However, I am not interested in that. I am interested in the cold hard comparisons of what makes each platform different. As a reference, I started writing this article with little to no knowledge of how to use either platform. I believe that this lack of knowledge gives me some form of a non-biased viewpoint on it. I will simply report on what I find and do a side by side comparison of the two frameworks.

I went out and compiled the information I could find on the two frameworks (And Angular 1.0). Below you will see what the results of this research show.

#The Basics

| Attribute | AngularJS | Angular 2 | React |
|-----------------------|------------------|-----------------|----------|
| *Version* | 1.5.0-rc1 / 1.49 | 2.0.0 - In Beta | 0.14.6 |
| *Author* | Google | Google | Facebook |
| *Language* | JavaScript/HTML | TypeScript | JSX |
| *Size* | 143k | 764k | 151k |
| *Github Stars* | 46.4k | 8.4k | 34.4k |
| *Github Contributors* | 1,386 | 189 | 604 |

This table is pretty straight forward, Angular is by Google and React is by Facebook.

#The Meta Stuff

| Attribute | AngularJS | Angular 2 | React |
|------------------------|--------------|--------------|--------------------|
| *Churn* | Reduced | Reduced | High |
| *Tooling* | Low | High | High |
| *Code Design* | JS into HTML | JS into HTML | JavaScript Centric |
| *JavaScript “Fatigue”* | Less | Less | More |

These tend to be high conversation areas that get started on Reddit and Hacker News.

#The Other Information

| Attribute | AngularJS | Angular 2 | React |
|---------------------------|--------------------|---------------------|--------------------|
| *DOM* | Regular DOM | Regular DOM | Virtual DOM |
| *Learning Curve* | High | Medium | Low |
| *Packaging* | Weak | Medium | Strong |
| *Abstraction* | Weak | Strong | Strong |
| *Debugging General* | Good HTML / Bad JS | Good JS/Good HTML | Good JS / Bad HTML |
| *Debug Line NO* | No | No | Yes |
| *Unclosed Tag Mentioned?* | No | No | Yes |
| *Fails When?* | Runtime | Runtime | Compile-Time |
| *Binding* | 2 Way | 2 Way | Uni-Directional |
| *Templating* | In HTML | In TypeScript Files | In JSX Files |
| *Component Model* | Weak | Strong | Medium |
| *Building Mobile?* | Ionic Framework | Ionic Framework | React Native |
| *MVC* | Yes | Yes | View Layer Only |
| *Rendering* | Client Side | Server Side | Server Side |

Most topics I have seen on Reddit, Hacker News, and other social media sites tend to only cover and talk about the first two tables. People get stuck on the thought of JavaScript Fatigue, JSX, TypeScript, Churn, and Size. I hope I have helped to break some of that by including all of the other information too which shows some holes in both frameworks.

I think some of the most interesting differences are the size, community, debugging, DOM, and mobile. When talking about mobile React has its own way of building mobile apps through React Native. Angular doesn’t try to tackle it’s own mobile and allows others like Ionic Framework to do this for it. 

While the AngularJS 1 community is very large, the Angular 2 community is just gaining ground. This is in part due to Angular 2 still being in beta, but the traction it has already gotten is pretty outstanding. React’s community continues to grow and become more diverse as well.

When looking at sizes, there are some distinct differences between React and Angular. While Angular 2 is currently at around 764k in size, they are going to be trimming a lot of that off in the coming months. React also does not do as much of the MVC model as Angular so it can get away with having a much smaller size. However if you add in the extras to React to make it fully MVC then you will still come out with a smaller framework size than Angular 2.

Debugging is sometimes the hardest thing to do. I think it is interesting how each framework handles debugging. Angular 2 and it’s runtime debugging tends to give you less information than React and it’s compile time debugging.

Finally, I wanted to talk a second about the Virtual DOM that React uses. This was a new concept to me, and I looked at it more in depth. This concept seems to be much faster since you don’t have to work with the heavyweight parts that the real DOM has. Manipulating and changing a virtual copy of your DOM that is lightweight, and then only pushing the changes that are shown when doing a diff on the DOM is pretty awesome.

After spending a few days researching and reading other articles on these two frameworks, I have found that there is a lot of give and take when it comes to comparing the two. On some things Angular is better and on others, React is better. As always you should asses the project that you are working on to find out what best suits your needs. Maybe this comparison chart can help you in that decision. 

For more information on this check out Pluralsight's libraries of videos that pertain to this. Some that come to mind are:

* [Building a Real-time App with React, Flux, Webpack, and Firebase](https://app.pluralsight.com/library/courses/build-isomorphic-app-react-flux-webpack-firebase/table-of-contents)
* [Building a Full-Stack App with React and Express](https://app.pluralsight.com/library/courses/react-express-full-stack-app-build/table-of-contents)
* [Building Mobile Apps With the Ionic Framework and AngularJS](https://app.pluralsight.com/library/courses/building-mobile-apps-ionic-framework-angularjs/table-of-contents)
* [Building a Web App with ASP.NET 5, MVC6, EF7 and AngularJS](https://app.pluralsight.com/library/courses/aspdotnet-5-ef7-bootstrap-angular-web-app/table-of-contents)

# The Other Guy

Those should get you started on the right track, and if you really want to throw a wrench in and learn something new then check out [RiotJS](http://riotjs.com/)

I can't list all other frameworks out there, so if you have one you like that isn't one of these then please comment below!

### About The Author:

Shannon Duncan is the Founder and Author at [Unrestricted Coding](http://unrestrictedcoding.com). He mentors and teaches people of all ages how to code and enjoy the art of programming. Find him on [twitter](https://twitter.com/TheUCofficial), [linked-in](https://www.linkedin.com/in/jsduncan98), and [github](https://github.com/shadowcodex).
