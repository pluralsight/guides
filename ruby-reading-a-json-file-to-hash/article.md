In Ruby read json file to hash can be achieved using File Handling. Example to parse json file to hash is illustrated with this tutorial.

## What is JSON?

If you are new to **JSON **(JavaScript Object Notation) you might want to read [Introduction to JSON](http://www.json.org/)

[![json](http://rubyinrails.com/wp-content/uploads/2014/04/json.gif)]

## Require JSON

You need to require JSON before JSON parse, thus let us require at first.

``` ruby
require 'json'
 => true
```

If above steps returns false then probably you don't have json gem installed on your machine. You can install JSON gem using following command,

``` ruby
 gem install json
```


### Step 1: Open JSON file for Parsing

You can create file handle to parse JSON file in Ruby with given command,

``` ruby
file = File.read('file-name-to-be-read.json')
```

The above command will open file ‘file-name-to-be-read.json’ in read mode.

### Step 2: Parse Data from File


``` ruby
data_hash = JSON.parse(file)
```

The above command will parse the data from the file using file handle created with name 'file' and variable data_hash will have parsed hash from the file.

## Access and Sample JSON File

File name - json_data_ruby_in_rails.json
Content:

``` json
{
 "title"	:	"Ruby In Rails",
 "url"	:	"http://rubyinrails.com",
 "posts"	:	{
     "1":"strftime-time-format-in-ruby",
             "2":"what-is-gemset"
            }
}
```


## After Executing read JSON file to hash code


``` ruby
 require 'json'
file = File.read('file-name-to-be-read.json')
data_hash = JSON.parse(file)

```

Now, the variable data_hash will be consisting of hash from which we can **access** values as,

``` ruby
data_hash['title']
 => "Ruby In Rails"
data_hash.keys
 => ["title", "url", "posts"]
data_hash['posts']
 => { "1" => "strftime-time-format-in-ruby", "2" => "what-is-gemset" }
```

This way you can access contents of Hash read from JSON file.

If you come across any problem while parsing JSON file, let us know through comments so that we can help you out.

## Reference

Read [JSON Implementation for Ruby](http://flori.github.io/json/) for more on JSON.

