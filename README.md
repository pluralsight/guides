# hack.guides()

Welcome to the hack.guides() content repository.  This repository contains published and unpublished versions of awesome technical guides written by our community.  You can browse all the guides right here or head over to our [companion site](http://www.pluralsight.com/guides) for a more focused reading experience.

## What is hack.guides()?

A community, movement, and website of full of developers with the [ambitious goal](http://tutorials.pluralsight.com/faq) of educating ourselves and the world.

## Join the movement

**You** have unique experience, and we want to help you share it.  We have several ways for you to contribute to [our mission](http://tutorials.pluralsight.com/faq) of educating the world.

1. [Write your own guide](http://tutorials.pluralsight.com/write/).
2. [Find a guide to review](http://tutorials.pluralsight.com/in-review) and help out the author by editing their work.
3. [Fork our Github-powered CMS](https://github.com/pluralsight/guides-cms) and help us improve the experience of creating and consuming great technical guides.
4. Fork this repository using the fork button above and send us a Pull Request with your edits.
5. [Join our slack community](https://hackguides.herokuapp.com/)
6. Tell your friends!

### What's in it for you?

We've all learned by reading great content on the web.  Now is your chance to give back.  Studies show the best way to learn is by teaching.  In fact, writing is a great way to teach and learn at the same time.

Sharing your knowledge is also a great way to promote your personal brand and build your reputation in the development community.

Looking for a new job?  Sharing great content with potential employers is the perfect way to prove yourself before even walking into the interview room.

### Repository organization

We are working hard to create a great reading experience on [our website](http://www.pluralsight.com/guides).  We've also taken care to neatly organize our content in this repository.  If you're a fan of [git](http://www.git-scm.com), Github, and the [CLI](https://en.wikipedia.org/wiki/Command-line_interface) you're in the right place.  From here you can clone or fork this repository and read all the guides in whatever environment you choose.

Here's an abbreviated look at how this repository is structured:


    |---- faq.md
    |---- published.md
    |---+ published
    |----   + c-c++
    |----   + ruby-ruby-on-rails
    |----   + python
    |----   +   + guide-1
    |----   +       article.md
    |----   +       details.json
    |
    |---- in-review.md
    |---+ in-review
    |----   + c-c++
    |----   +   + guide-2
    |----   +       article.md
    |----   +       details.json
    |----   + ruby-ruby-on-rails
    |----   + python
    |
    |---- draft.md
    |---+ draft
    |----   + c-c++
    |----   + ruby-ruby-on-rails
    |----   +   + guide-3
    |----   +       article.md
    |----   +       details.json
    |----   + python

There are main 3 directories, published, in-review, and draft.  Each of these directories corresponds to how complete a guide is.  Guides in the 'in-review' folder are currently undergoing editing from authors and our community editors.  Guides in the 'draft' folder are in-progress and not ready for external reviewers yet.

#### Reading

You can browse the content right here.  Start by picking whether you want to read published, in-review, or draft guides.  Then pick the corresponding file, [published.md](https://github.com/pluralsight/guides/blob/master/published.md), [in_review.md](https://github.com/pluralsight/guides/blob/master/in_review.md), or [draft.md](https://github.com/pluralsight/guides/blob/master/draft.md).  These files will give you an overview of all the guides in that stage.

You can also browse the directories for each publish stage.  Each publish stage folder has another list of folders organized by topic or stack.  So, you can easily skip to the categories that are most interesting to you.

Finally, inside each category is another list of directories, 1 for each guide.  Each guide is organized in a single directory named after the title of the guide.  In each directory, you'll find a file named `article.md`.  Clicking on that file lets you read the content directly on github.com.

#### Editing

Got an idea for improving the article? Click the fork button in the top-right corner of the article, edit it directly on github.com, and send us a Pull Request with your contribution.  No software installation necessary!

#### Want to easily browse the guides?

1. Click on the [published.md file](https://github.com/pluralsight/guides/blob/master/published.md) to see our published guides.
    - Here you can find links to our top guides as well as links to read more from your favorite contributors.
2. Click on the [draft.md file](https://github.com/pluralsight/guides/blob/master/draft.md) to search through drafts of potential guides. 
3. Click on the [in_review.md file](https://github.com/pluralsight/guides/blob/master/in_review.md) to search through completed drafts that are undergoing final evalutation and review.

## Creating a new guide the hacker way

We're hackers too and there's nothing like creating content from your own environment.  Feel free to clone this repository, create your own directory with `article.md` and `details.json` files inside the `draft` or `in-review` folders, and send us a Pull Request.

We're constantly working on improving this experience.  Soon we'll have an easy-to-use script to automatically setup the guide for you. All you worry about is writing great content.

Keep checking back for that, but until then copy an existing `details.json` file, fill it in with your information, and send us a Pull Request.  We're happy to walk you through it!
