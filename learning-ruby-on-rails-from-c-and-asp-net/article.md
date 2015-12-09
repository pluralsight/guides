One of the things I've been doing lately is digging into Ruby on Rails. I've always wanted to learn Rails since I was first exposed to Rails at an AjaxWorld conference in '06 (at least, I think in 06). David Heinemeier Hansson actually presented!

Alas, I never could devote enough time to get past the tipping point. I'm comfortable with C# and ASP.NET and evolving those skills has always been the priority. But the perfect storm has happened recently- in order to save space in my apartment, I got rid of my PC desktop and now only use my MacBook. I got tired of using Parallels and Visual Studio, and a new project came up in which I could either use ASP.NET MVC (which I'm calling .MVC from now on) or RoR. I thought it was time to finally try RoR.

The verdict is Rails is great. I still can't "express" myself as well as I want to with Rails, but in comparison to .MVC Rails is pretty slick, and the evolution of Rails (specifically, [combining Merb with Rails](http://rubyonrails.org/merb)) is pretty exciting.  Most importantly, learning Rails has actually made me a better C#/ASP.NET MVC developer- if you're working with .MVC you have to spend at least an afternoon playing around with Rails- you'll get an invaluable perspective on MVC and web programming.

Now, a discussion on any programming language/framework always causes a heated debated.  I'm not an expert (or even a beginner) on Ruby or Rails, but these are my initial impressions.

### Ruby, as a Language

One major leap between .MVC and Rails is Ruby as a language. Yes, Ruby and C# are both OO languages, but Ruby is a [dynamic language](http://en.wikipedia.org/wiki/Dynamic_programming_language)- and if you're up on C# 4.0, you'll know that C# is on it's way to [becoming a dynamic language too](http://ironpython-urls.blogspot.com/2008/12/c-becomes-dynamic-language.html). So if you want keep your C# skills on the cutting edge, get a head start on a full fledged dynamic language!

I originally made the mistake of jumping in and assuming Ruby was more vb- or js- esque than it actually was.  It's pretty smart in the way it behaves, almost a tailored version of C#.  Ruby's use of [symbols](http://glu.ttono.us/articles/2005/08/19/understanding-ruby-symbols) is pretty big difference over other languages that's extensively used in Rails and pretty handy.

Lambda expressions are also core part of Ruby, and are becoming a more integral part of C#.  This allows Ruby to be extremely concise in getting things done- which is awesome when you know what you're doing.  When you don't it can be confusing.  But lambdas make sense and are awesome when you know how to use them- and knowing Ruby can help you grasp lambda expressions in C#.

Rails leverages Ruby's dynamic language to make a lot happen under the hood.  I personally found letting the Rails framework do its thing as one of the biggest hurdles in learning Rails.  I wanted to either program or explicitly orchestrate everything!  One prime example is the persistence model in Rails-  I struggled to figure out how properties where set in models from migration classes- but it's entirely automatic!  Also, a lot of methods are created dynamically.  This allows an extremely fluid expression in writing code, making Ruby a pretty natural programming language.  (Example: the find_by ActiveRecord methods; declaring links like edit_xxx_path).

C# is making its way into being a more fluid programming language similar to Ruby.  Meaning, writing code and describing code are converging to be the same thing.  This is seen in lambda expressions and fluent interfaces, where you can daisy chain methods together.  [Moq](http://code.google.com/p/moq/) is a good example of the fluent interface approach.  I wouldn't be surprised if there's even a larger convergence with C# and Ruby in the years to come.

### Rails, as a Framework

Those in the .NET world who follow [Jeremy Miller](http://codebetter.com/blogs/jeremy.miller/) have probably heard the term of [opinionated software.](http://codebetter.com/blogs/jeremy.miller/archive/2008/10/23/our-opinions-on-the-asp-net-mvc-introducing-the-thunderdome-principle.aspx) He talks about opinionated software in the context of [FubuMVC](http://code.google.com/p/fubumvc/), an alternative to .MVC written in an opinionated style.  It's worth checking out- it's very interesting to use but Rails still cracks the MVC shell wide open because it leverages Ruby's language features.

Opinionated software originally came from the Adam and Eve of Rails, [37Signals](http://gettingreal.37signals.com/ch04_Make_Opinionated_Software.php).  Rails is highly opinionated- which is a blessing and curse.  The great thing about Rails is that it does what it does extremely well.  The MVC triangle are completely separate but seamlessly integrated, and extension points are explicit.  It's how MVC should be- and .MVC doesn't come close to Rails as an MVC framework.  Hate NHibernate configuration?  You've never seen an ORM framework until you've used ActiveRecord.  Really want to abandon code-behind files and excessive code in views? Really want to do TDD (and even BDD)?  Rails right now is the iPhone in a tin cup and string world.

Here are some highlights:

1. Respond_to method for rendering html, xml, json or whatever from a single controller.  .MVC ActionResult could learn a thing or two.
2. Rails' partial layouts, partial actions and partial views are pretty slick compared .MVC's master pages, partial views, and partial layouts.  A prime example is using Rails partial views to render a collection of objects.
3. Passing data between controllers and actions is a lot slicker than .MVC ViewData
4. Rails helpers are a lot more helpful.
5. Rails is pretty slick when it comes to mapping between posted data and Models- a lot better than binders.
6. And of course, Rails' database integration will make you wonder why you ever spent more than 5 minutes learning about databases- it's such a model centric approach with migrations you'll hate doing any data tier work in .MVC.  (Although learning databases are extremely important no matter what language/framework you use).
7. Plugins.  Rails Plugins are simply awesome- extremely fluid integration into your Rails application.  And combining plugins and other support with gem makes maintenance and upgrades a breeze (forget the GAC!)

### Where .MVC Shines

I've been hyping up Rails a lot, but there is a major disadvantage with Rails: good luck straying from the Rails track.  If you get off the train, you're walking alone.  This is something .MVC does well- allows you to change the game with whatever architecture pattern you wish.  It's one reason I love .NET- you can do literally do whatever want (yes, you can whatever you want with any language, but the .NET framework is pretty awesome).  .MVC is extremely extensible- from replaceable view engines, routing engines, controller factories, coupled with whatever architecture pattern you want.  And all very testable.  C#'s usage of extension methods also add pretty nice extensibility to classes, too.

In fact, Rails' plans to integrate with Merb as way to be more modular is a prime example of something .MVC already does well- allows you to mix and match what you want instead of forcing conformance.  Allowing total and explicit control over the application stack- including your architecture- is a great advantage in maintaining the health of ongoing software. Yes, you can evolve Rails to do what you want, but the transparency .NET offers is something I don't see in Rails just yet.  It's definitely a pro and a con- on the one hand, Rails offers fluid interaction between components.  On the other, .NET doesn't force you into any specific pattern.

### In Conclusion

The bottom line is, check out Rails.  The best way to get started is with [the Rails guides](http://guides.rails.info/).  From a .MVC perspective, you'll learn a lot about MVC and what an MVC framework can do- and it will help you out in your .NET development.  Knowing Ruby will also keep your C# skills sharp and help you in becoming a well rounded developer.  I'll even bet you carry over some Rails tricks to your next .MVC app!

