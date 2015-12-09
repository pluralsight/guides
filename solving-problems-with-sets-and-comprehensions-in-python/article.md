I see many newcomers to Python struggle to solve problems in a simple and elegant way. This isn't because they're stupid or poor programmers, but because they come from languages that lack these high-level data structures and functions.

When I teach Python classes, I try to expose my students to the many tools that the language provides, through examples and exercises. Many of these examples come from real-life problems that I have solved, either for my consulting clients or for my own needs. I have found that forcing people to think in this way, through repeated exercises, is one of the best ways to introduce someone to Python, and to its many capabilities.

In this blog post, I'll walk you through a real-life problem that I recently had, and the ways in which Python allowed me to solve it, using some of Python's most useful commands, paradigms, and functionality. I hope that by walking you through several potential solutions, and then the one I finally used, you will gain some insights into how to use Python to solve your own problems.

What I present here is the sort of exercise that I like to give people I teach and mentor -- and is also an example of what I've put in my new ebook, "Practice Makes Python," a collection of 50 exercises and solutions to help you improve your Python skills.

Some background: I recently decided to move a popular community mailing list (3,000 subscribers, 60-80 postings/day) from my server to Google Groups. I asked people to joint he Google-based list themselves, and added many others myself, as the list manager. However, after nearly a week, only half of the list had been moved. I somehow needed to learn which people on the old list hadn't yet signed up for the new list.

Fortunately, Google will let you export a list of members of a group to CSV format. Also, Mailman (the list-management program I was using on my server) allows you to list all of the e-mail addresses being used for a list. Comparing these lists, I think, offers a nice chance to look at several different aspects of Python, and to consider how we can solve this real-world problem in a "Pythonic" way.

# Inputs and outputs

The goal of this project is thus to find all of the e-mail addresses on the old list that aren't on the new list. The old list is in a file containing one e-mail address per line, as in:


``` csv
person1@domain1.com
person2@domain2.com
person3@domain3.com
```

The output that I want is a subset of this list, also with one e-mail address per line. However, any e-mail address on this old list should not be on the new list. The new list comes in a CSV (comma-separated values) file that looks like:


``` csv
1234928448@netvision.net.il,"1234928448",member,,"email","moderated -override",2014,11,23,23,31,6,America/Los_Angeles
abcdlife@gmail.com,"Joe Schmoe",member,,"no email","moderated - override",2014,11,24,5,50,6,America/Los_Angeles
12xylgil@gmail.com,"John Doe",member,,"no email","moderated - override",2014,11,23,9,31,8,America/Los_Angeles
```

For the sake of clarity, let's call the old list "old.txt", and the new list "new.csv".

The first goal will be to get the e-mail addresses from new.csv into a format that we can use for comparison with old.txt. Most Python developers realize that we can iterate over the lines of a file by treating the file object as an iterator. That said, I can display every line of a file with:


``` python
for line in open('new.csv'):
    print line
```

Of course, we're not really interested in displaying the entire line. Rather, we want to get just the e-mail address from each line. Fortunately, we can treat a CSV file as a set of records separated by commas. Thus, we might naively think that we can use the "str.split" operator to turn the line (a string) into a list:

``` python
for line in open('new.csv'):
    print line.split(",")
```

That will show each of the lines in the file as a list. The first element of that list will be an e-mail address, which means that we can print all of the e-mail addresses in the file simply by saying:

``` python
for line in open('new.csv'):
    print line.split(",")[0]
```

# Using a list comprehension

There are several problems with this approach, though: First of all, if we're dealing with CSV data, then we should almost certainly use the "csv" module in the Python standard library. Not only is it likely to execute faster than our "for" loop, but it knows how to handle things like quotation marks in CSV fields.

Moreover, if we read the contents of new.csv using the "csv" module, we have more flexibility of working with the data. That's because "csv.reader" returns an iterator -- an object that knows how to behave inside of a loop or similar context. We will take advantage of this later on. But for now, if I want to get a list of the e-mail addresses, I can do something like this:


``` python
import csv
list(csv.reader(open('new.csv')))
```

The thing is, I now have a list of the records in "new.csv". I'm not really interested in the entirety of each record, but only in the first element of each one. In other words, I want to iterate over each line in new.csv, grabbing the first field. I could do the following:

``` python
import csv
new_addresses = []
for record in csv.reader(open('new.csv')):
    new_addresses.append(record[0])
```

The above will work, with the e-mail addresses in the "output" list. but there is an even nicer way to do this, using Python's list comprehensions. A list comprehension allows us to apply a function or expression on each element of an iterable, and returns the resulting list. We can thus turn the above three lines into one line:

``` python
import csv
new_addresses = [record[0] for record in csv.reader(open('new.csv'))]
```

This is a classic use of list comprehensions in Python: I take an iterable, run a transformation on it, and then get a list back.

# Moving to sets

I want to compare the old e-mail addresses with the new ones, which means that I'm going to be doing a lot of comparisons and lookups. We can imagine, for example, doing something like this:

``` python
not_in_new = [ ]
for old_address.strip() in open('old.txt'):
     if old_address not in new_addresses:
         not_in_new.append(old_address)
```
In other words, we'll iterate over the lines of old.txt, removing any leading or training whitespace (e.g., the final newline character) from each record. Then we'll grab each of the old addresses, and see if it is in the new list. If not, then we add it to the list "not_in_new".

While this code will work, it doesn't feel right, mostly because of the use of "not in" on a list. If your list is short, then using "in" or "not in" is just fine. But if your list is long, then "in" on a list is slow and inefficient. In computer science terms, we would say that it takes O(n) time, meaning that the average lookup time will be proportional to the length of the list. Given that we will be looking through the "new" list many times, it would be smarter for us to use an index of some sort, in order to speed up the lookup time.

Fortunately, Python provides us with such a data structures -- the "set", which is part of the Python standard library. You can think of a set as a dictionary without values (i.e., only the keys). This means that the values must be hashable, meaning that they must be immutable. Each value can only exist once in the set. But sets are perfect for lists of usernames, words, filenames, IP addresses, and other lists in which elements must remain unique and for which you want fast lookups.

Even better, we can create a set of the new addresses with only slight changes to our list comprehension:

``` python
import csv
new_addresses = set([record[0] for record in csv.reader(open('new.csv'))])
```

The above code creates a list (thanks to our list comprehension), and then uses it to create a set. But in modern versions of Python, we don't need to pass our list comprehension to a set. Rather, we can use curly braces, for a "set comprehension":

``` python
import csv
new_addresses = {record[0] for record in csv.reader(open('new.csv'))}
```

Now, new_addresses is a set, with O(1) lookup time. It doesn't matter (that much) how many elements are in new_addresses; lookups will happen in constant time, and will be relatively speedy.

However, we're still stuck with the fact that our old addresses were loaded into a list. If we can load those into a set, we can then compare the sets using built-in operators. Fortunately, this is fairly easy to do with our set comprehension:

``` python
import csv
new_addresses = {record[0] for record in csv.reader(open('new.csv'))}
old_addresses = { address.strip() for address in open('old.txt') }
```

Now that we have the addresses in sets, we need to find out what old addresses haven't yet been subscribed to the new list. We could use a "for" loop, but we can take advantage of the "difference" method, which allows us to compare two sets in precisely this way. That will return a new set -- which, as an iterable, can then be passed to the str.join method. If I join the difference with '\n' (newline) characters, and then write the resulting string to a new file, I get the list of addreses for which I was working:

``` python
with open(output_filename, 'w') as f:
     f.write('\n'.join(old_list.difference(new_list)))
```

If you aren't familiar with the "with" statement in Python, it allows us to create a new object and assign it to a variable. The file is only open within the "with" block.

The final program that I used was as follows:

``` python
import csv

old_list = { line.strip() for line in open('old.txt') }
new_list = { record[0] for record in csv.reader(open('new.csv')) }

with open('output.txt', 'w') as f:
     f.write('\n'.join(old_list.difference(new_list)))
```

The beauty of this solution, from my perspective, is that I didn't have to write any new functions, create any classes, or do anything particularly special. All I needed was to combine built-in data structures -- files and sets, in this case -- with set comrehensions and some set methods, and I was able to get a working solution within minutes.

I hope that this was useful and interesting. My ebook, "Practice Makes Python," contains 50 such exercises, with soultions and explanations. You can get a 10% discount on the book by using the coupon code "HACKHANDS" when you check out. You can learn more about the book at [http://lerner.co.il/practice-makes-python](http://lerner.co.il/practice-makes-python) . If you have any questions or comments, please contact me via e-mail at reuven@lerner.co.il. I would be delighted to hear from you!

