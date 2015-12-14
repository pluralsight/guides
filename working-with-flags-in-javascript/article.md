In development there is a concept called flags. They can be used for lots and lots of different things. The use case we are going to talk today about is protecting data. 

Sometimes you want to be able to protect a variable in certain situations. Today we are going to talk about a use case in Javascript where we want to retain the data inside a div that we replaced.

## The Problem

Grey Elerson is working on a pretty cool project, and I don't want to give away too much information than that. Within this project he has attached comments to an object, and wanted to have a hover effect on each comment. The problem came with the hover command in jQuery. The mouseenter event would fire twice before firing the exit event.

His original code looked like this:

```javascript
$('div.comment').hover(
    function() { 
        var high = $(this).height();
        var wide = $(this).width();
        comment_html = $(this).html();
      
        console.log(comment_html);
      
        var hoverstring = "<div style='height: "+high+"px ; width: "+wide+"px ;'><p style='font-size: 30px; color: #efefef; text-align: center;position: relative; top: 50%;transform: translateY(-50%); opacity: 0.5;'>Reply</p></div>";
      
        $(this).html(hoverstring).css("background", "#444");
        console.log("mouse in");
    }, 
    
    function(){
        $(this).css("background", "#efefef").html(comment_html);
        console.log("mouse out");
    }
);
```

So he was calling an event on hover to replace the div's contents with a "reply to this comment" message. Very simple idea but it was breaking. When he was hovering on the item it sometimes fired the hover event twice right in a row. Well since his originall comment HTML was not protected he just overwrote it with the new html. This mean that when he stopped hovering on the comment the reply message still showed.

So to fix this we need to protect the original comment.

## The solution

We discussed several solutions to this, and the first couple of solutions were just complex and bloated. The included a json array with the original comment HTML stored, but this involved loops and lots of data being generated. We also would have to decide if the solution needed to be on the server side or the client side.

The solution we came up with addressed the initial problem. Double firing of the mouseenter event and no protection.

Here is my solution broken down.

```javascript
var comment_html;
var commentflag = false;
var comment_id;
````

So what is going on in those three lines is very important. Let's break it down even more.

1. We create a comment_html variable to hold the original div contents
2. We create a flag called commentflag. This is just a boolean variable that will tell us if we have a comment stored (true) or not (false). We start off with a value of false because we don't have a comment stored when start the page.
3. We create a comment id variable. This will hold the name of the comment we are protecting. We want to store the id of the comment before we set the commentflag to true.

Next I changed up the jQuery even to an on event. This is a personal style preference and I believe it creates better code readability.

```javascript
  $('div.comment').on("mouseenter", function(){
      if(!commentflag) 
      {
        comment_html = $(this).html();
        console.log(comment_html);
        var hoverstring = "<div style='height: 100%; width: 100%;'><p style='font-size: 30px; color: #efefef; text-align: center;position: relative; top: 50%;transform: translateY(-50%); opacity: 0.5;'>Reply</p></div>";
        $(this).html(hoverstring).css("background", "#444");
        console.log("mouse in");
        comment_id = $(this).attr('id');
        commentflag = true;
      } else {
        // Do nothing because we have a comment saved...
      }
  });
```

Now let's break this code block down as well:

1. We use an if statement to check if we have a comment that is protected. When the if statement evaluates to true (AKA commentflag == false) then we do not have a comment stored.
2. Once we are inside the if statement then we go ahead and store this comment.
3. Logout this for debug purposes
4. Save the hoverstring (The contents to override this div's html with)
5. Go ahead and replace the html in this div
6. Set comment_id equal to this div's id attribute. This helps us remember what comment is stored.
7. Finally we want to protect this comment so we set commentflag = true; This sets the comment as protected.

So this code will follow a logic of: Do I have a comment protected? If yes then go ahead and do nothing because we don't want to hurt the protected comment. If no then go ahead and start the process for showing the reply to comment message and save the current contents to protect them. 

Thus if a mouseenter event is fired twice the first time it will store the comment, and the second time it will do nothing. It can fire that mouseevent as much as it wants, and it will never hurt the protected comment.

Ok so now how do we handle the mouse out event? Easy, like this...

```javascript
$('div.comment').on("mouseleave", function(){
    if($(this).attr('id') === comment_id) 
    {
      $(this).html(comment_html).css("background", "#efefef");
      console.log("mouse out");
      commentflag = false;
    } else {
      // do nothing because we don't like you...
    }
  });
```

Let's break this down too:

1. ON the mouseleave event we want to start off with an if statement.
2. The if statement evaluates to true if the id of the div firing the mouseleave event is the same div that protected itself.
3. If it evaluates as true then we want to go ahead and restore the old html data, and set the commentflag back to false.

This logic makes sure that if for some reason we are firing a mouseleave event from another div than what the mouseenter event was called, that we don't accidentally overwrite the wrong div with the original content. This also sets the commentflag back to false allowing a new div to be hovered on.

That concludes the fix. Simple yet works well. Enjoy!

### About The Author:
![Shannon Duncan](https://pbs.twimg.com/profile_images/672481536826937344/GeAx6xl4_200x200.jpg)

Shannon Duncan is the Founder and Author at [Unrestricted Coding](http://unrestrictedcoding.com). He mentors and teaches people of all ages how to code and enjoy the art of programming. Find him on [twitter](https://twitter.com/bikemybodyback), [linked-in](https://www.linkedin.com/in/jsduncan98), and [github](https://github.com/shadowcodex).


