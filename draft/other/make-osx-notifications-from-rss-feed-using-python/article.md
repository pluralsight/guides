
Today we are going to see how make OSX notifications from RSS Feed of any website using Python.

So first of all, let us see how to make a simple notification. We are going to use <a href="https://en.wikipedia.org/wiki/AppleScript">AppleScript</a> for this.
Go to terminal and Type:

    >>osascript -e 'display notification "Lorem ipsum dolor sit amet" with title "Title"'

We shall see a notification popping up in right up corner like this: 

<img src="https://dl.dropboxusercontent.com/u/31435562/notification_SS.png">

Now, we will use this command to pop up notifications using python from rss feeds.

Now, we are going to use <a href="https://pypi.python.org/pypi/feedparser">feedparser</a> library to get feed from a url. For example:

    import feedparser 
    feeds = feedparser.parse('https://feedity.com/ruddra-com/VFZQWlFU.rss')  

    print([(x.title, x.id) for x in feeds.entries])


It will return a list of tuples containing urls and titles from the feed.

Now we will make an infinite loop and inside the loop we will continuously call the rss feed (will also pause for few moments, no worries) and show notifications:


    while True:
        d = feedparser.parse(rss_url)
        for ds in d.entries:
           print (d.id, d.title)
           time.sleep(5)

Now lets build the command for applescript which will put pop ups.

    apple_cmd = "osascript -e '{0}'"

    for ds in d.entries:
        base_cmd = 'display notification "{0}" with title "{1}"'.format(ds.title, "Foobar")
        apple_cmd.format(base_cmd)

Thats all we need. Now if we print the `apple_cmd`, we shall see commands like this:

    osascript -e 'display notification "Foobar Title" with title "Foobar"'


Now using python's `subprocessor` module, we will call these commands:

    import subprocessor

    subprocessor.Popen([apple_cmd], shell=True)


Also there is one more thing, we need to remove duplicate entries from feed, we don't want to see same notifications twice, or see old notifications on and on and on. So, we will check if the feed is updated like this:

    update_time = d.feed.updated

We will verify if `update_time` is greater than previous iteration in while loop. Also, we will store `feed id` in a list so that we can check if new feed has ids which was already stored in previous iteration.

Finally, the code will look like this:

    last_updated_time = None
    while True:
        d = feedparser.parse(rss_url)
        updated_time = d.feed.updated
        if updated_time == last_updated_time:
            print('No new feed')
        else:
            last_updated_time = updated_time
            for entry in d.entries:
                _id = entry.id
                if _id in dup_ids:
                    print('Entry already exists')
                else:
                    dup_ids.append(_id)
                    base_cmd = 'display notification "{0}" with title "{1}"'.format(_notification, rss_title)
                    cmd = apple_cmd.format(base_cmd)
                    subprocess.Popen([cmd], shell=True)
                time.sleep(5)
    time.sleep(5)

Thats it, we will see popups of new notifications if feed is updated.

The full working code is here: https://github.com/ruddra/AppleFeedNotifier


And don't forget see more blogs at my site http://ruddra.com.

Cheers!
