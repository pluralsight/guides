Elasticsearch _(ES)_ is a powerful Full Text Search Engine based on [Apache Lucene][lucene].
A key characteristic of Elasticsearch is that it’s distributed at it's core, meaning that you can easily scale it horizontally for the purpose of redundancy or performance. Elasticsearch can also be used as data store engine, but it has some disadvantages:

* __Security__ - Elasticsearch does not provide any internal security or access
control system. The only way to protect ES from external access is with a firewall.
* __Computation__ – There is limited support for advanced computation on the
database side.
* __Data Availability__ – Data in Elasticsearch is available in "near real time", -
meaning that if you submit a comment to a post and refresh the page, it might not
show up as the index is still updating.
* __Durability__ – ES is distributed and relatively stable, but
backups are not as high priority as in other data store solutions. This is an important consideration when Elasticsearch is your primary data store.

For the purpose of this tutorial we’ll cover some of the Elasticsearch
basis, so you can later understand what’s happening behind the scenes. To fully
grasp how it works and how to use its API, take a look at the [Elasticsearch
Documentation][es-docs].

ES is document oriented; it stores its entire objects
inside documents. It also indexes these documents to make them searchable.
A document belongs to a type and a type belongs to an index. You can draw some
parallels to how a traditional relational database is structured:

```
Relational DB  ⇒ Databases ⇒ Tables ⇒ Rows      ⇒ Columns
Elasticsearch  ⇒ Indices   ⇒ Types  ⇒ Documents ⇒ Fields
```

Each ES Index can be split into multiple pieces called shards. This
is done if, for example, your required storage volume exceeds the abilities of a
single node (server), or could be used to parallelize operations across shards
to increase the performance of your ES cluster.

Note that the number of primary shards is fixed the moment an index is created. This
effectively defines the maximum amount of data stored in your index, as each node
is limited by the amount of CPU, RAM and I/Os it can have to fit at least one shard. You can only change the amount of replica shards after the index has been created, though that only affects the throughput of your cluster and not the actual data storage capabilities. You can read more about that on [Elasticsearch Scale Horizontally][es-scale].

It’s important to understand these basic concepts, as we will configure some of them
later in this post.

Making a model searchable
-------------------------

I've provided an example app using Ruby on Rails, which can be found on my GitHub
[itay-grudev/es-tuturial](es-tutorial). Take a look at it and you’ll notice that it contains an entire
app with search, highlighting of matches and search suggestions based on the
[term suggester][es-term-suggester].

The official [elasticsearch][es-gem] gem for Ruby contains the most important
libraries for using Elasticsearch with Ruby. But we’ll use the
[elasticsearch-model][es-model-gem] gem, which is built on top of the `elasticsearch`
gem and contains all the tools to make a model searchable.

To get started, make sure you add the gem to your Gemfile:

```ruby
gem 'elasticsearch-model'
```

### Settings and indexing

Let's assume that we have a model called `Article` with two columns, `title` and
`body` of types `string` and `text`, correspondingly.

The first thing we need to do is include the `Elasticsearch::Model` functionality
inside our model. Additionally, the `Elasticsearch::Model::Callbacks` ensures
that Elasticsearch indexes are updated when a record is created or updated.

```ruby
class Article < ActiveRecord::Base
  include Elasticsearch::Model
  include Elasticsearch::Model::Callbacks

  index_name Rails.application.class.parent_name.underscore
  document_type self.new.class.name.downcase
end
```

Note the `index_name` and `document_type` calls. (I prefer to have an index_name
with the name of my application and document type named after my model.) To set up the document type we need to specify to ES how we want to index our data.

```ruby
settings index: { number_of_shards: 1 } do
  mapping dynamic: false do
    indexes :title, analyzer: 'english'
    indexes :body, analyzer: 'english'
  end
end
```

Notice the `number_of_shards` setting I’m using. I've set it to 1 since I was
only using a single machine and a very small amount of data (< 100MB). But if you have a very high amount of data or would like to increase the performance of ES, increase the number of primary shards. Since both fields will contain text written in English I’m using the `english` analyzer. Elasticsearch has a wide set of Analyzers including [Language Analyzers][es-lang-analysers]. Depending on the data stored within a field, a different analyzer may give better results. For example, a keyword analyzer is useful for data like zip codes and ids, among other things.

You can also specify the type of the field as basic types, such as an integer
requiring much simpler indexing methods. So, assuming that you have a field `status`,
which can only take the values of `[0, 1, 2, 3]` indicating whether the article
is a draft, private, unlisted or public, you’ll specify it like this:

```ruby
indexes :status, type: :byte
```

Or in the case of an author id in a column called `user_id` you will do this:

```ruby
indexes :user_id, type: :integer
```

Elasticsearch also supports complex types as `Arrays`, `Objects` or `Nested`, which
is an array of objects. There is support for geographic coordinates and IP addresses, among other things. Take a look at the full set of [Types][es-types] Elasticsearch supports
when setting up your index.

If your model also has additional fields that you don't need to store/index on
ES, you could save space by overriding the default `as_indexed_json`
method and choose which fields are included in the JSON representation of a record that is sent to ES.

```ruby
def as_indexed_json(options = nil)
  self.as_json( only: [ :title, :body ] )
end
```

Additionally, if you’d like to index an aggregate field or otherwise a virtual
field (one that is not stored on your database). you could do this:

```ruby
def as_indexed_json(options = nil)
  self.as_json( only: [ :title, :body ], methods: :method_name )
end
```

This is where `:method_name` is either a symbol or an array of symbols corresponding to
method names in your model. It’s useful if, for example, your page body is stored
in a format as Markdown. You can have a method that renders Markdown as plain text
and then send that plain text to Elasticsearch.

Note that your document can have more fields than those specified in your
`settings` block, but they won't be indexed.

### Implementing the `self.search()` method

 The final and most complex addition to your model is the `self.search`
method that will spawn requests to Elasticsearch. It’s the hardest method
to write, as you’ll need to have some knowledge of the
[Elasticsearch Query DSL][es-query-dsl].

What I’m about to show you is universal to most projects, but if the functionality you need isn’t covered here, check out the [Elasticsearch Query DSL][es-query-dsl].

You’ll notice that queries to ES are written in JSON. In Ruby, JSON
is easily represented with a `Hash`, so we’ll use a method that will call
the Elasticsearch cluster and send it our query written as a Ruby `Hash.` For the
sake of brevity, I’ll show how the method looks and will describe each
section in a separate code block.

```ruby
def self.search(query)
   __elasticsearch__.search(
   {
     query: {
        multi_match: {
          query: query,
          fields: ['title^5', 'body']
        }
      },
      # more blocks will go IN HERE. Keep reading!
   }
 end
end
```

We spawn a Multi Match query to ES with the search term as the first
argument of the class method and we specify which fields we’re running this
query on. Additionally, we’re specifying the weight of matches inside each
field. In the example, the `title` field is five times as important as the
`body` field. There are several types of Multi Match queries, specified by the
`type` parameter. Here are the possible options:

| Type Parameter  | Description                                                |
| --------------- | ---------------------------------------------------------- |
| `best_fields`   | __(default)__ Finds documents which match any field, but uses the _score from the best field. |
| `most_fields`   | Finds documents which match any field and combines the _score from each field. |
| `cross_fields`  | Treats fields with the same analyzer as though they were one big field. Looks for each word in any field. |
| `phrase`        | Runs a match_phrase query on each field and combines the _score from each field. |
| `phrase_prefix` | Runs a match_phrase query on each field and combines the _score from each field. |

For more details on each option see [Elasticsearch Multi Match Query][es-query-multi-match].

The next section I’ll add is for highlighting the matched keywords inside the
text. We want to match both fields, `title` and `body`, and encapsulate them in
a `<mark>` tag.

```ruby
highlight: {
  pre_tags: ['<mark>'],
  post_tags: ['</mark>'],
  fields: {
    title: {},
    body: {},
  }
},
```

The next thing I’d like to set up is the `term` suggester, so if the user misspelled a given keyword, ES would propose recommendation for improving the
search term.

```ruby
suggest: {
  text: query,
  title: {
    term: {
      size: 1,
      field: :title
    }
  },
  body: {
    term: {
      size: 1,
      field: :body
    }
  }
}
```

This enables search term suggestions based on both the `title` and `body`
fields with a [Hamming Distance][hamming-distance] of up to `1` (you can change that). You can read more about the [Elasticsearch Term Suggester][es-term-suggester]
or take a look at the other [Elasticsearch Suggesters][es-suggesters]. For example, there’s a [Completion Suggester][es-completion-suggester] for auto complete functionality.

Let’s go back to the scenario in which we had a `status` field of type integer taking
the values of `[0, 1, 2, 3]` indicating whether the article is a draft, private,
unlisted or public. Assuming that `status == 3` means that an article is public,
we might want to specify a filter in our search to only show public articles.

```ruby
filter: {
  terms: {
    status: [ 4 ]
  }
},
```

And that sums it up for the model. Now, let’s head to the visualization
of the data returned from ES.

Setting up Controllers and Views
--------------------------------

Thanks to the neat search method in the model the controller will simply
look like this; where the `if` statement is there only to ensure the controller doesn't
fail if the search form is submitted empty.

```ruby
class SearchController < ApplicationController

  def search
    unless params[:query].blank?
      @results = Page.search( params[:query] )
    end
  end

end
```

Because the `self.search()` method returns a result of type
`Elasticsearch::Model::Response` instead of `ActiveRecord::Relation`, but they
are similar and somewhat compatible, with a little bit of tricky handling you can
make your view support both. This is useful if you’d like to have your
search page work with `ActiveRecord` queries as well, without having to use
another view.

Assuming that the results from `Elasticsearch`/`ActiveRecord` are stored in the
`@results` variable, you’ll always need to run `@results.respond_to? :es_method`
and verify that the ES method exists before you call it. This is
how you’ll maintain compatibility with `ActiveRecord`.

```slim
- @results.each do |page|
  article
    h3 = page.title
    - if page.respond_to? :highlight
      p = raw page.highlight.body.to_a.join(' ')
    - else
      p = raw truncate(page.body, length: 300)
```

To show the suggestions proposed by ES I use a helper method to
aggregate suggestions and build a new string, based on every suggestion with
rating above `0.65`. This is how my helper looks:

```ruby
def autosuggest_aggregate(response, fields, query)
  # Stores unique words and their autocorrect suggestions
  words = { }

  # Iterate over fields
  fields.each do |field|
    # Iterate over query words
    response.send(field).to_a.each do |word|
      # If any options are available
      if word.options.length > 0
        # Append word if it doesn't already exist
        new_word = false
        unless words.include? word.text
          words[word.text] = { text: '', score: nil, freq: nil }
          new_word = true
        end
        word.options.each do |option|
          if new_word or words[word.text][:score] * words[word.text][:freq] < option[:score] * option[:freq]
            words[word.text][:text] = option[:text]
            words[word.text][:score] = option[:score]
            words[word.text][:freq] = option[:freq]
          end
        end
      end
    end
  end

  new_query = query.dup

  # Generate a new string based on the high score suggestions
  words.each do |word, suggestion|
    if suggestion[:score] > 0.65
      new_query[word] = suggestion[:text]
    end
  end

  # Return the new query or false if there were no modifications
  return new_query unless new_query == query
  false
end
```

Once you have this, the term suggestions in your view would simply look like this:

```slim
- if @results.respond_to? :response and (suggestion = autosuggest_aggregate(@results.response.suggest, [:title, :body], params[:query]))
  p
    | Search instead for:
    = link_to suggestion, search_path( query: suggestion )
```

A little homework
-----------------

I've prepared a simple [demo application][es-tuturial] that demonstrates Elastic
Search integration with Ruby on Rails. You can take a look at how I implemented it, or try to
tweak it into doing more advanced things. To get it running: install all gems, create the database (SQLite 3), run an `elasticsearch` instance in the background and seed the Application with the provided Wikipedia articles within the `/db/seed/` directory.

```bash
bundle install
rake db:migrate
elasticsearch >/dev/null </dev/null &
rake db:seed
```

After you've got it running go to [localhost:3000](http://localhost:3000/) and
try searching for ruby or language. Also try combinations of words to see
how different articles are sorted: ruby gem or ruby gemstone.

Now search for `gem` and try changing the Multi Match Query Type Parameter to
`phrase_prefix` to see what will change.

If you still think this is exciting, how about going through the
[Elasticsearch Query DSL][es-query-dsl]. Be amazed and experiment!

About the author
----------------

Itay Grudev is a student currently pursuing a degree in _Computer Science and Physics at the University of Aberdeen, United Kingdom_.

Itay is mostly interested in Linux, Security, Electronics and Amateur Radio. He loves Open Source and Free Software. He is a talented developer and one of those people spending time to change your `i++` to `++i`, crazy about efficiency and beautiful code. His favorite technologies are `C++`, `Qt` and `Ruby on Rails`.

[lucene]: https://lucene.apache.org/
[es-gem]: https://github.com/elastic/elasticsearch-ruby
[es-docs]: https://www.elastic.co/guide/en/elasticsearch/guide/master/getting-started.html
[es-scale]: https://www.elastic.co/guide/en/elasticsearch/guide/current/_scale_horizontally.html
[es-types]: https://www.elastic.co/guide/en/elasticsearch/reference/current/mapping-types.html
[es-model-gem]: https://github.com/elastic/elasticsearch-rails/tree/master/elasticsearch-model
[es-suggesters]: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters.html
[es-query-dsl]: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html
[es-query-multi-match]: https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-multi-match-query.html
[es-lang-analysers]: https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-lang-analyzer.html
[es-term-suggester]: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-term.html
[es-completion-suggester]: https://www.elastic.co/guide/en/elasticsearch/reference/current/search-suggesters-completion.html
[hamming-distance]: https://en.wikipedia.org/wiki/Hamming_distance
[es-tuturial]: https://github.com/itay-grudev/es-tutorial

