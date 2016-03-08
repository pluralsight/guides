Nested Attributes is a feature that allows you to save attributes of a record
through it's associated parent (within it's form).

In this example we will consider the following scenario. We are making an online
store with lots of products. Each `Product` can have zero or more `Variants`s.
Each variant represents a variation of the same product, but in different color
for example. Both have a `name` and `price`. Each product will also be
associated with one `Image` record, containing a `url`, `alt` and a `caption`.

Further in the article we will be improving these models:

```ruby
class Product < ActiveRecord::Base
  has_many :variants
  has_one :image
  # Attributes: name:string, price:float
end
```

```ruby
class Variant < ActiveRecord::Base
  belongs_to :product
  # Attributes: name:string, price:float
end
```

```ruby
class Image < ActiveRecord::Base
  belongs_to :product
  # Attributes: url:string, alt:string caption:string
end
```

One to one association
----------------------

The simplest example of Nested Attributes is with a one-to-one association. To
add Nested Attributes support to the product all you need to do is add the
following line. What it does is that it would proxy the saved attributes from
the `Product` model to the `Omage` model.

```ruby
class Product < ActiveRecord::Base
  has_many :variants
  accepts_nested_attributes_for :image
end
```

In the Product form you will need to add the additional fields for the image
association. You would do it by using the `fields_for` helper.

```slim
= form_for @product do |f|

  // Product attributes
  .form-group
    = f.label :name
    = f.text_field :name  
  .form-group
    = f.label :price
    = f.text_field :price

  // Image attributes
  = f.fields_for :image do |f|
    = f.label :url
    = f.text_field :url

    = f.label :alt
    = f.text_field :alt

    = f.label :caption
    = f.text_field :caption

  = f.submit
```

The only part left is to modify the controller to accept those new attributes.
The entire idea behind Nested Attributes is that you won't have to add
additional code in the controller to handle this input and the association, but
you do need to allow those attributes to reach the model, which is what
[Strong Parameters][strong-params] would prevent by default. So you will need to
add the following to the `product_params` method in the `ProductsController`.

```ruby
def product_params
  params.require(:product).permit(
    :name, :price,
    image_attributes: [ :id, :url, :alt, :caption ]
  )
end
```

And voila! Now you can edit the `Image` association of the `Product` model
inline from the same form. Now let's proceed and look at how we will build
the same behavior with a many to many relationship.

Many to many association
------------------------

As product variants are quite simple (2 fields), there is really no point in
creating a separate page for editing them. Instead we would like to edit them
inline from the same product form along with the product attributes. As each
product can have many variants that means that we will have to handle more than
one item and also adding new variants and deleting old ones. Let's address the
problems one by one.

### Displaying multiple associations

Well actually, the `fields_for` block kindly does that for us, but bare with me,
the following is actually a requirement for the next section.

We are going to create a new partial, called `_variant_fields.slim` containing
just the variant fields, like this:

```slim
= f.label :name
= f.text_field :name

= f.label :price
= f.text_field :price
```

And back in the product form to render the fields we will just take advantage of
the fact that `fields_for` yields a block for each association:

```slim
= f.fields_for :variants do |f|
  = render 'variant_fields', f: f
```

### Adding new associations

Now for adding new associations we will need to create some JavaScript that
adds new fields. What I like to do, is have a link that when you click on it,
will add a new pair of fields. Something like:

```slim
= link_to_add_fields 'Add Product Variant', f, :variants
```

I've wrote this really useful helper method, that will create a link with the
`data-form-prepend` containing the entire contents of the `_variant_fields.slim`
file. The idea is that when you click on it you will use some simple re-usable
JavaScript to append those fields to the end of the form.

Now the actual helper looks quite complex and messy but bare with me. It is very
simple as most of the code just handles the arguments and the key logic rests in
the last 7 lines. Place this code in your `application_helper.rb`.

```ruby
def link_to_add_fields(name = nil, f = nil, association = nil, options = nil, html_options = nil, &block)
  f, association, options, html_options, name = name, f, association, options, block if block_given?

  options = {} if options.nil?
  html_options = {} if html_options.nil?

  if options.include? :locals
    locals = options[:locals]
  else
    locals = { }
  end

  if options.include? :partial
    partial = options[:partial]
  else
    partial = association.to_s.singularize + '_fields'
  end

  # Render the form fields from a file with the association name provided
  new_object = f.object.class.reflect_on_association(association).klass.new
  fields = f.fields_for(association, new_object, child_index: 'new_record') do |builder|
    render(partial, locals.merge!( f: builder))
  end

  # The rendered fields are sent with the link within the data-form-prepend attr
  html_options['data-form-prepend'] = raw CGI::escapeHTML( fields )
  html_options['href'] = '#'

  content_tag(:a, name, html_options, &block)
end
```

On the JavaScript side I use a simple jQuery to find every element with the
name attribute set to `new_record` and replace it with a timestamp. This solves
a problem when you are adding more than one new record, both will have the same
id.

```javascript
$('[data-form-prepend]').click( function(e) {
    var obj = $( $(this).attr('data-form-prepend') );
    obj.find('input, select, textarea').each( function() {
      $(this).attr( 'name', function() {
        return $(this).attr('name').replace( 'new_record', (new Date()).getTime() );
      });
    });
    obj.insertBefore( this );
    return false;
  });
```

### Deleting associations

Fortunately the `accepts_nested_attributes_for` has some neat features for
deleting associations. If we pass the `allow_destroy: true` argument to
`accepts_nested_attributes_for` it will destroys any members from the attributes
which contain a `_destroy` key.

```ruby
accepts_nested_attributes_for :variants, allow_destroy: true
```

This could be implemented with a simple checkbox in your view. So I added this
to my `_variant_fields.slim`:

```slim
= f.check_box :_destroy
= f.label :delete
```

### Modifications to the Model and Controller

Again, as before the additions to the `ProductsController` are just in the
`product_params` method which now should also include the `variants_attributes`.

```ruby
def product_params
  params.require(:product).permit(
    :name, :price,
    image_attributes: [ :id, :url, :alt, :caption ],
    variants_attributes: [ :id, :name, :price, :_destroy ]
  )
end
```

The `Product` model just has the following addition, to enable Nested Attributes
for the `variants` association

```ruby
accepts_nested_attributes_for :variants, reject_if: :all_blank, allow_destroy: true
```

Notice the `reject_if :all_blank` option I used. It designates that any
record where all of the values are blank (excluding any value for `_destroy`)
should be rejected. `reject_if` also supports a Proc which can be used for some
validation.

There are two other very useful options. The first one is the `limit` option
which specifies the maximum number of records that can be processed.

The second option is `update_only` which applies to one-to-one associations and
has a rather interesting behavior. If it is set to true, it will only update the
attributes of the associated record. If set to false upon change it wont touch
the old record but will create a new one with the new attributes. By default it
is false and will create a new record unless the record include an `id`
attribute, which is exactly the reason why we included the `id` attribute in the
`product_params` for the one-to-one image association. An alternative solution
would have been to define the nested attributes like this:

```ruby
accepts_nested_attributes_for :image, update_only: true
```

You can read more about [Nested Attributes][nested-attrs] in the Ruby on Rails
documentation where each of the configuration options is demonstrated and very
well documented.

About the author
----------------
Itay Grudev is a student currently pursuing a degree in _Computer Science and
Physics at the University of Aberdeen, United Kingdom_.

Itay is mostly interested in Linux, Security, Electronics and Amateur Radio. He
loves Open Source and Free Software. He is a talented developer and one of those
people spending time to change your `i++` to `++i`, crazy about efficiency and
beautiful code. His favorite technologies are `C++`, `Qt` and `Ruby on Rails`.

[strong-params]: http://edgeapi.rubyonrails.org/classes/ActionController/StrongParameters.html
[nested-attrs]: http://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html
