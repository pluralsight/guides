<p>Today we are going to see how make OSX notifications from RSS Feed of any website using Python.</p>

<p>So first of all, let us see how to make a simple notification. We are going to use <a href="https://en.wikipedia.org/wiki/AppleScript">AppleScript</a> for this. <br>
Go to terminal and Type:</p>

<pre><code>&gt;&gt;osascript -e 'display notification "Lorem ipsum dolor sit amet" with title "Title"'
</code></pre>

<p>We shall see a notification popping up in right up corner like this: </p>

<p><img src="https://dl.dropboxusercontent.com/u/31435562/notification_SS.png"></p>

<p>Now, we will use this command to pop up notifications using python from rss feeds.</p>

<p>Now, we are going to use <a href="https://pypi.python.org/pypi/feedparser">feedparser</a> library to get feed from a url. For example:</p>

<pre><code>import feedparser 
feeds = feedparser.parse('https://feedity.com/ruddra-com/VFZQWlFU.rss')  

print([(x.title, x.id) for x in feeds.entries])
</code></pre>

<p>It will return a list of tuples containing urls and titles from the feed.</p>

<p>Now we will make an infinite loop and inside the loop we will continuously call the rss feed (will also pause for few moments, no worries) and show notifications:</p>

<pre><code>while True:
    d = feedparser.parse(rss_url)
    for ds in d.entries:
       print (d.id, d.title)
       time.sleep(5)
</code></pre>

<p>Now lets build the command for applescript which will put pop ups.</p>

<pre><code>apple_cmd = "osascript -e '{0}'"

for ds in d.entries:
    base_cmd = 'display notification "{0}" with title "{1}"'.format(ds.title, "Foobar")
    apple_cmd.format(base_cmd)
</code></pre>

<p>Thats all we need. Now if we print the <code>apple_cmd</code>, we shall see commands like this:</p>

<pre><code>osascript -e 'display notification "Foobar Title" with title "Foobar"'
</code></pre>

<p>Now using python's <code>subprocessor</code> module, we will call these commands:</p>

<pre><code>import subprocessor

subprocessor.Popen([apple_cmd], shell=True)
</code></pre>

<p>Also there is one more thing, we need to remove duplicate entries from feed, we don't want to see same notifications twice, or see old notifications on and on and on. So, we will check if the feed is updated like this:</p>

<pre><code>update_time = d.feed.updated
</code></pre>

<p>We will verify if <code>update_time</code> is greater than previous iteration in while loop. Also, we will store <code>feed id</code> in a list so that we can check if new feed has ids which was already stored in previous iteration.</p>

<p>Finally, the code will look like this:</p>

<pre><code>last_updated_time = None
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
</code></pre>

<p>Thats it, we will see popups of new notifications if feed is updated.</p>

<p>The full working code is here: <a href="https://github.com/ruddra/AppleFeedNotifier">https://github.com/ruddra/AppleFeedNotifier</a></p>

<p>Cheers!</p>