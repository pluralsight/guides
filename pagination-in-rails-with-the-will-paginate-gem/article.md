Pagination in Rails can be implemented easily with the will_paginate gem. The will_paginate gem modifies Collection of ActiveRecord in a way to implement pagination.

## What is will_paginate?

* It is ** Pagination Library ** for Rails
* **Querying for Records** and ** Displaying Page Links ** can be automated using this gem
* Source: [ Github - will_paginate ](https://github.com/mislav/will_paginate)

Steps for integration of will_paginate with Rails -

### Step 1: Add to Gemfile

Add `gem will_paginate` in your **Gemfile**


``` ruby
gem 'will_paginate'
```


This will ensure **latest version of gem 'will_paginate'** being used with your Rails Application

### Step 2: Bundle Install

Perform **Bundle Install** of your Rails Project

``` ruby
bundle install
```


This command needs to be executed in Rails Application directory which will install gem 'will_paginate' in your Ruby environment if already not installed. If this gem is already available in your environment then it will just use the gem will Rails Application.

### Step 3: Code Changes for Basic Integration

Suppose, we have User Model in database and we are displaying all Users in show.erb file
And we need pagination for the same. This can be implemented in following way -

#### Controller - Action changes

Add pagination parameter to the Model which is being queried for the Paginated result set.

#### Why?
It adds necessary parameters in the resultant collection of record to display pagination links in front-end (.erb). Parameters that are added:

* **current_page** - the page number for current paginated result data set
* **total_entries** - numbers of records in database satisfying given criteria
* **limit** - per page limit for the paginated result data
* **offset** - current paginated data set -> to show current page


``` ruby
 class UserController

  def index
    @users = User.paginate(:page => params[:page], :per_page => 5)
  end

end
```


##### Parameters:

* `:page` - This is parameter sent in Query String. Based on this, which records are to be fetched is decided. i.e. ** Offset is decided ** .
* `:per_page` - This is ** Number of results ** that you want to fetch per page i.e. From offset **_ERB - show.erb_**

``` ruby
# Display Each result from @users - Paginated Result

# Then add in the pagination links like this
<% will_paginate @users %>
```


This is just displaying the paginated result on front-end. For Awesome UI Elements on front-end you can refer [Semantic UI Rails Integration](rubyinrails.com/2014/03/how-to-integrate-semantic-ui-rails-with-rails-application/)

You can inspect ** @users  ** by debugging and you will find  ** current_page,  total_entries, limit, offset **  with the variable ** @users ** that help in displaying pagination links by will_paginate as given below.

### Step 4: Adding Pagination Links

It will add pagination links like, `← Previous 1 2 3 4 5 6 7 8 9 … 817 818 Next →` on front-end. Number of Pages here 817, 818 is decided based on total numbers of records that you have in your database.

## Extra Configurations

### Adding Custom Parameters in will_paginate AJAX Request

You can add custom parameters in AJAX request send by will_paginate for getting paginated result. You can do this like:

``` ruby
 <% {:is_active: true} %>
```


This will send is_active=true as a parameter in the query string.
Query string would become as http://someurl.com/?page=2&is_active=true
This way you can send multiple custom parameters with AJAX request of will_paginate

### Performing Javascript/JQuery Operation before AJAX request

Sometimes you might need to perform some Javascript/JQuery operation(E.g. Showing loader while the request fetches result from Backend) before you send AJAX request to get paginated result. This can be achieved by,
Enclose the pagination code of erb in div tag, like


``` ruby
<% {:is_active: true} %>
```


Then you can use will_paginate_id for performing javascript operation like,


``` javascript
$('#will_paginate_id').bind('click', function(event){

  window.scrollTo(0,0);

  // Some code to show loader on screen

  $('#loading').html('Loading');
});
```


### Conclusion - Pagination in Rails

* We illustrated the easiest way of implementing Pagination in Rails using will_paginate gem
* **Let us know through comments** if you face any difficulties implementing the same or any other problem while using the gem for implementing pagination in rails.

