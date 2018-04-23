Sending basic JSON data made out of strings to an API (Application Programming Interface) is, in most cases, a simple and straightforward task. But what about sending files that consist of numerous lines of binary data in various formats? Such data require a slightly different approach for sending files to the API.

In this guide we will examine the two main approaches of handling file uploads, **multipart form data** and **base64 encoding**, through a Rails 5 API application using both the [paperclip](https://github.com/thoughtbot/paperclip) and the [carrierwave](https://github.com/carrierwaveuploader/carrierwave) gems.

## Approaches for sending files to a Rails 5 API 
### Multipart form data
  Multipart forms have been around since [HTML 4](https://www.w3.org/TR/html401/interact/forms.html#h-17.13.4). They were introduced because the standard *application/x-www-form-urlencoded* forms did not handle bigger amounts of data sufficiently well. 
  
  According to [W3C](https://www.w3.org/), *multipart/form-data* is used for forms that "contain files, non-ASCII data, and binary data." What multipart forms do differently is that they break the data into different chunks  representing the various characteristics of the file (size, content type, name, contents) and the characteristics of the form's standard text and number data.
  
  To make things clearer, let's consider the following form:
  
  
```html
   <form action="http://localhost:3000/api/v1/items"
         enctype="multipart/form-data"
         method="post">
   <p>
   What is your name? <input type="text" name="submit-name"><br>
   What file are you sending? <input type="files" name="file"><br>
   </p>
   <input type="submit" value="Send"> <input type="reset">
 </form>
 
```
If the user enters "John" in the text input, and selects the text file "file1.txt", the browser will send back the following data:

```
   Content-Type: multipart/form-data; boundary=AaB03x

   --AaB03x
   Content-Disposition: form-data; name="submit-name"

   John
   --AaB03x
   Content-Disposition: form-data; name="files"; filename="file1.txt"
   Content-Type: text/plain

   ... contents of file1.txt ...
   --AaB03x--

```
Every part of the form is separated by a boundary, which represents a different string (<code>AaB03x</code> in the example). The part itself contains binary infomation with a <code>Content-Type</code>. Larger files can be broken down in chuncks and assembled in the server, enabling a file to be streamed and to have its integrity maintaned in cases of connection interruprtion.

Let's consider another file. If the user selects another file  "file2.gif", the browser will construct the parts as follows:

```
   Content-Type: multipart/form-data; boundary=AaB03x

   --AaB03x
   Content-Disposition: form-data; name="submit-name"

   John
   --AaB03x
   Content-Disposition: form-data; name="file"
   Content-Type: multipart/mixed; boundary=BbC04y

   --BbC04y
   Content-Disposition: file; filename="file1.txt"
   Content-Type: text/plain

   ... contents of file1.txt ...
   --BbC04y
   Content-Disposition: file; filename="file2.gif"
   Content-Type: image/gif
   Content-Transfer-Encoding: binary

   ...contents of file2.gif...
   --BbC04y--
```

Here, it can be seen that there is another part added to the <code>form-data</code>. This time, since the file is not in a <code>text/plain</code> format, it is broken down in binary, as it can be inferred from the <code> Content-Transfer-Encoding</code> property. 

The <code>Content-Type</code> property gives information about the type of the file, also known as its [media (MIME) type](https://en.wikipedia.org/wiki/Media_type). If the <code> Content-Type </code> property  is not defined and  the file is not in text format, its format will default to <code> application/octet-stream </code>, which means that the final file will be binary and without a type. 

**When sending data to an API, it is always good to include a <code>Content-Type</code> to each part which contains a file, otherwise there is no way to validate the contents of the file.**


### Base64 encoding
Base64 for is one of the most commonly used **binary to text** encoding formats. It uses an algorithm to break up binary code in pieces and convert it in ASCII characters (text). It has a wide array of applications - apart from being used to encode files into text in order to send them to an API, it is also used represent images as a content source in CSS, HTML and SVG. 

The structure of base64 encoded files is very simple. It consists of two parts - a MIME type (similar to the multipart <code>Content-Type</code> and the actual base64 encoded string:
```
data:image/gif;base64,iVBORw0KGgoAAAANag...//rest of the base64 text
```
When it comes to uploading, the file is encoded into base64 on the client and decoded on the server. The base64 string can be easily attached to a JSON object's attribute:
```javascript
 {
  "file": {
    "name": "file2",
    "contents": "data:image/gif;base64, iVBORw0KGgoAAA..."
  }
 }
```

An API would easily be able to pick up the parameter and decode it back to binary. This makes base64-encoded file uploads convenient for APIs, since the format of the message containing the file and the way it is transferred does not differ from the standard way information is sent to an API.


### Base64 encoding vs. Multipart form data

Base64-encoded files can be easily used by JSON and XML APIs since they are represented as text and can be sent through the standard <code> application/json</code> format. However, the encoding increases the file size by 33%, making it difficult to transfer larger files. Encoding and decoding also adds a computational overhead for both the server and the client. Therefore, base64 is suitable for sending images and small files under 100MB. 

Multipart forms, on the other hand, are more "unnatural" to the APIs, since the data is encoded in a different format and requires a different way of handling. However, the increased performance with larger files and the ability to stream files makes multipart form data more desirable for uploading larger files, such as videos.

## Setting up Rails API for file upload

 Let's apply these concepts. First, let's set up a Rails 5 API application and scaffold a model that is going to be used. 
 
### Installing and configuring Rails 5 API
 
To create a Rails 5 API, you need Ruby 2.2.4 installed. After you have a suitable Ruby version, the first step is to install the newest version of Rails through your terminal/command prompt:

```bash
gem install rails --pre --no-ri --no-rdoc
```

This will give you the ability to run <code> rails new </code> using the most recent version (currently 5.0.0.beta3):

```bash
rails _5.0.0.beta3_ new fileuploadapp --api
```

This will create a brand new Rails 5 app, and with the <code> --api </code> option included in the generator, it will be all set up to be used as an API. Move to the directory of the new project:

```bash
 cd fileuploadapp
```

Go to the Gemfile and uncomment <code>  jbuilder </code> and <code> rack-cors </code>.
```ruby
# Gemfile.rb
 gem 'jbuilder', '~> 2.0' 
 gem 'rack-cors'
```   

[JBuilder](https://github.com/rails/jbuilder) is used to create JSON structures for the responses from the application. In MVC terms, the JSON responses are going to be the view layer (AKA UI layer or presentation layer) of the application. Therefore, all Jbuilder-generated responses have to be put in <code> app/views/(view for a particular controller action) </code>. 

[rack-cors](https://github.com/cyu/rack-cors) enables cross-origin resource sharing ([CORS](https://developer.mozilla.org/en-US/docs/Web/HTTP/Access_control_CORS)). Simply put, rack-cors will enable browser-based applications (AngularJS, React) and mobile applications to request information from the API. Go to the application configuration file and add the configurations for CORS:
```ruby
#config/application.rb
module Fileuploadapp
  class Application < Rails::Application
    config.middleware.insert_before 0, "Rack::Cors" do
      allow do
        origins '*'
        resource '*', :headers => :any, :methods => [:get, :post, :options]
      end
    end
    config.api_only = true
  end
end
```
 This configuration will give full access to the API (<code>*</code> means that everything is accepted). This is not a problem for this guide, since we are going to work only with a local server.
 
Install the gems:
```bash
bundle install
```
With the gems installed, the model is going to be scaffolded with Jbuilder-generated views and be ready to be consumed by client-side applications.
```bash
rails g scaffold Item name:string description:string
```
A model named <code>Item</code> is going to be scaffolded which has a name and description as strings. Note that there is still no information about the file - file information will be included in the model's schema at a later stage. For now, we are all set to go ahead and migrate the model into the database:

```bash
rails db:migrate
```
This will create a table in the database for the new model. Note that in Rails 5 you can use <code> rails </code> instead of <code> rake </code> for executing a migration command.
 

 **To continue with the guide, you have to choose the gem with which you would like to implement your file uploads**. If you would like to do both, I recommended that you use version control such as [git](https://git-scm.com/) so that the implementations don't clash.
 
##  File upload using Paperclip
To get started with Paperclip, first add the gem to your Gemfile:
 
 ```ruby
 #Gemfile.rb
 gem "paperclip", "~> 5.0.0.beta1"
```
  And install it:
 ```bash
 bundle install
```  
### Uploading a single file 


First, generate a migration that will add the attachment to the database. ** You have to put the same name for the attachment in the model as the one you put in the generator (in this case, it is  <code>picture</code>).**
 
```bash
 rails g paperclip item picture
```  
 After you are done, don't forget to migrate the database:
```bash
 rails db:migrate
```   
Second, add the following lines in the file of the model:
 ```ruby 
 #app/models/item.rb
class Item < ApplicationRecord
  has_attached_file :picture, styles: { medium: "300x300>", thumb: "100x100>" }, default_url: "/images/:style/missing.png"
  validates_attachment :picture, presence: true
  do_not_validate_attachment_file_type :picture
end

```

Here is what each method is about:
 1. <code>has_attached_file</code> is the main method for adding a file attachment . The first argument is the attribute of the model that is going to be used for the file attachment (In this case it is<code>:picture</code>, as we know from the database migration). <code>styles:</code> is an **optional** parameter that is going to distribute the uploaded files in different folders according to their file size. In this example, medium photos will be 300x300 pixels and up and will be put into the <code>/images/medium</code> directory. <code> default_url </code> is also an **optional** parameter used to specify the path of a default image that will be returned if the object from the model does not have an attached image. You can put the default image in <code> app/assets/images/(medium or thumb)/</code>. If you are uploading files that are different from images, you may omit the optional arguments.

 2. <code>validates_attachment</code> can validate the **content-type**, **presence**, and **size** of the file. You can see how the syntax for all the validations [here](http://www.rubydoc.info/github/thoughtbot/paperclip/Paperclip%2FValidators%2FClassMethods%3Avalidates_attachment). In this example, only the presence is checked.
 
 3. <code> do_not_validate_attachment_file_type</code> is used as there are [issues](https://github.com/thoughtbot/paperclip/issues/1429) with the file type validations of paperclip.
 
Third, add a permitted parameter that is going to accept <code>:picture</code>:
 
```ruby 
#app/controllers/items_controller.rb
def item_params
  params.require(:item).permit(:name, :description , :picture) # Add :picture as a permitted parameter
end
```
  
The last step is to add the <code>:picture</code> to the Jbuilder view, so that when we <code>GET</code> an item, it will return its picture:
```ruby   
json.extract! @item, :id, :name, :description, :picture, :created_at, :updated_at
```  

### Uploading multiple files 

#### Attaching a document model to item
To make uploading multiple files using Paperclip possible, **add another model that is related to the main model and that will contain the files.** In the current example, let's add the capability for an item to have multiple PDF documents. First, let's generate a model:

```bash
 rails g model document item:references
 rails g paperclip document file

```
The first generator is going to produce a model named <code>document</code> with a name as a string. <code>item:references</code> will generate a <code> belongs_to :item </code> reference which means that the item and the document will have a [one-to-many](http://www.databaseprimer.com/pages/relationship_1tox/) relationship. The second generator will generate fields for the attachment.

A reference also has to be added to the <code> Item </code> model as well:
```ruby  
class Item < ApplicationRecord
  #app/models/item.rb
  has_many :documents
  #...
end
```
Migrate the changes to the database schema:
```bash
 rails db:migrate
```
With this step finished, the <code> document </code> model is ready to be configured. Let's add the necessarry methods and validations to add PDF documents.
```ruby 
 #app/models/document.rb
 class Document < ApplicationRecord
  belongs_to :item
  has_attached_file :file
  validates_attachment :file, presence: true, content_type: { content_type: "application/pdf" }
end
```
This time, <code> validates attachment </code> checks if the document's type is <code> application/pdf </code>. 

With this addition, the <code> document </code> model is ready. Next, things need to be handled in the controller in order to handle creation of multiple files.

#### Creating an item with documents
When a new item is created, another parameter must get sent. <code>document_data</code> will contain an array of data about each document, either in multipart or in base64 format:

```ruby
    #app/contollers/items_controller.rb
    def item_params
      params.require(:item).permit(:name, :description , :document_data => []) #Add :documents_data in permit() to accept an array 
    end
```
Add the logic for creating multiple files in the controller:
```ruby
  #app/controllers/items_controller.rb
   #...
  def create
    @item = Item.new(item_params)

    if @item.save
      @item.save_attachments(item_params) if params[:item][:document_data]
      render :show, status: :created, location: @item
    else
      render json: @item.errors, status: :unprocessable_entity
  end
    #...
  end
```
In order to get <code>:document_data</code> into the model, a <code>attr_accessor</code> method needs to be added to the model:
```ruby
#app/models/item.rb
class Item < ApplicationRecord
  attr_accessor :document_data
end
```

After the item is successfully saved, the <code> document_data </code> parameter has to be checked. If the document data array is not empty, the controller will call the <code>save_attachments</code> that will attempt to create the documents. Here is how the model method has to look like:
```ruby
#app/models/item.rb
class Item < ApplicationRecord
  #...
  def save_attachments(params)
    params[:document_data].each do |doc|
      self.documents.create(:file => doc)
    end
  end
  #...
end
```
The <code> save_attachments </code> method is going to go through all the items in the <code>document_data</code> array. For each document, there is going to be an object from the document model created.


All done! Now it is possible to add multiple files to a document.
Add <code>:documents</code> to the Jbuilder view so that the documents get returned together with the item.
```ruby   
 #app/views/items/show.json.jbuilder
json.extract! @item, :id, :name, :description, :picture, :documents, :created_at, :updated_at
```  

### base64 upload
Suppose you want to create items and documents by sending in base64-encoded strings instead of form data. There are several additional steps that should be done for this to be achieved. As mentioned before, base64 involves much more overhead computation work and much more memory, but it is more easily integrated into the API.

#### Modifying the item 

First, the <code>attr_accessor</code> method will be used to get the base64 encoded string in the model:
 ```ruby
  #app/models/item.rb
class Item < ApplicationRecord
 attr_accessor :image_base
     #...
end
```
Second, add a private method for decoding <code>image_base</code> and assigning it to the <code> picture </code> attribute of the item.


```ruby 
  #app/models/item.rb

  private
    def parse_image
    image = Paperclip.io_adapters.for(image_base)
    image.original_filename = "file.jpg"
    self.picture = image
  end

``` 
The <code>parse_image </code> method takes <code>image_base</code> and puts it into Paperclip's [IO adapters](http://www.rubydoc.info/gems/paperclip/Paperclip/AdapterRegistry#registered_handlers-instance_method). They contain a registry which can turn the base64 string into binary. Because the file name is not stored, you can either put an arbitrary value (like "file.jpg", even if your file is not in jpg format) or add another <code>attr_accessor</code> for the name itself. Finally, <code> self.picture = image </code> assigns the image to the current instance of the object.

The method is ready, but it has to be called every time a new object is created. Let's add a filter that will call the <code> parse_image </code> method when that happens:

```ruby 
 #app/models/item.rb
class Item < ApplicationRecord
  before_validation :parse_image
  attr_accessor :image_base
  #...
end
```

The <code>before_validation</code> method will ensure that the base64 string will be parsed and assigned to the object instance before Paperclip validates the size, content type, and the rest of the attributes of the file.
  
In order to get <code>image_base</code>, it has to be entered as a parameter in the controller:
```ruby 
#app/controllers/items_controller.rb
def item_params
  params.require(:item).permit(:name, :description , :image_base , :document_data => []) #change :picture to :image_base
end
```
#### Modifying the documents
 Similarly to the item, the document data has to be decoded before being saved. So instead of sending a <code>:file </code>  directly, the <code> :file_contents </code> have to be sent:
 
```ruby
   #app/models/item.rb
   def save_attachments(params)
    params[:document_data].each do |doc|
      self.documents.create(:file_contents => doc) #change from :file to :file_contents
    end
  end
```

 Similarly to the item model, add <code>attr_accessor</code> for the <code> :file_contents </code> and a method to decode the base64-encoded content:
```ruby 
 #app/models/document.rb
 class Document < ApplicationRecord
  belongs_to :item
  before_validation :parse_file
  attr_accessor :file_contents
  has_attached_file :file
  validates_attachment :file, presence: true, content_type: { content_type: "application/pdf" }


  private

  def parse_file
    file = Paperclip.io_adapters.for(file_contents)
    file.original_filename = "pdfile.pdf"
    self.file = file
  end
end
```
##  File upload using CarrierWave
Add the Carrierwave gem to your Gemfile:
 
 ```ruby
 #Gemfile.rb
gem 'carrierwave'
```
Run the installation:
 ```bash
 bundle install
```  

### Uploading a single file
 CarrierWave isolates the logic of the uploaded files from the logic of the models using uploaders. To generate an uploader for the picture of the Item model, run the following command:
 ```bash 
 rails g uploader Picture
 rails g migration add_picture_to_items picture:string
``` 
The first generator will create a new directory in the application - <code> app/uploaders </code>. This is where all the uploaders are going to be stored. The second generator will add a picture column as string in the <code> Items </code> table which will store a reference to the file.


Don't forget to migrate the database:
 ```bash 
 rails db:migrate
``` 

In the generated uploader, you can see various configuration options. Right now we need the file type validation option. Since the uploader is used to upload pictures, the content types of each picture need to be whitelisted (or approved):
 ```ruby
 #app/uploaders/picture_uploader
  def extension_white_list  
    %w(jpg jpeg gif png)
  end
``` 

Now, mount the uploader to the item model:
 ```ruby
class Item < ApplicationRecord
  mount_uploader :picture, PictureUploader
end
``` 
 When a new model instance is created, the uploader will automatically associate the <code> picture </code> with it. The picture will contain the url with the image, which can be reached through <code> Item.picture.url</code>.
 
 Next, enter the <code> :picture </code> parameter as a parameter:
```ruby 
 #app/controllers/items_controllerr.rb
 def item_params
  params.require(:item).permit(:name, :description :picture) # Add :picture as a permitted paramter
 end
```

Last, add the <code>:picture</code> to the Jbuilder view, so that when we <code>GET</code> an item, the API will return its picture:
```ruby   
 #app/views/items/show.json.jbuilder
json.extract! @item, :id, :name, :description, :picture, :created_at, :updated_at
```  
### Uploading multiple files
 Let's add PDF documents to the item. This will require adding another uploader as well as a model named document belonging to the item. To accomplish this task, run these two generators:
 
```bash 
 rails g model document item:references document:string
 rails g uploader Document

```
 The first generator creates the <code>document</code> model with references to the <code> Item </code> and to a <code> file </code> column that gives a reference to its CarrierWave file. The second generator creates the uploader in the <code> app/uploaders</code> directory.
 Add the new <code> document </code> model in the database schema:
```bash 
rails db:migrate
```
Mount the uploader in the <code> document </code> model:
```ruby
 #app/model/document.rb
class Document < ApplicationRecord
  belongs_to :item
  mount_uploader :document, DocumentUploader
end
```
 Because the documents will be PDF only, adjust the whitelist accordingly in the uploader:
 
```ruby 
#app/uploaders/document_uploader.rb
def extension_white_list
    %w(pdf)
end
``` 
In the <code>Item</code> model, add the following lines:
```ruby 
 #app/model/item.rb
class Item < ApplicationRecord
  mount_uploader :picture, PictureUploader
  has_many :documents
  attr_accessor :document_data
end
```
<code>has_many :documents</code> adds a relation one-to-many between the item and the documents. <code> attr_accessor :document_data </code> will let us send extra attributes to the controller. This attribute is going to contain an array with data about every PDF document.

We are done with the model! Let's proceed to the controllers:

1. Add the <code> document_data </code> array as a permitted parameter. _Array parameters must always be the last parameter_:
 ```ruby 
    #app/controllers/item_controller.rb
    def item_params
      params.require(:item).permit(:name, :description, :picture, :document_data => []) #add document_data as a permitted parameter
    end
```
2. Add a loop that will create documents after the item is saved:
 ```ruby 
 #app/controllers/item_controller.rb
  def create
    @item = Item.new(item_params)

    if @item.save
      #iterate through each of the files
      params[:item][:document_data].each do |file|
          @item.documents.create!(:document => file) 
          #create a document associated with the item that has just been created
      end
      render :show, status: :created, location: @item
    else
      render json: @item.errors, status: :unprocessable_entity
    end
  end
```

Finally, update your Jbuilder view, so that the documents get returned together with the item:
```ruby   
 #app/views/items/show.json.jbuilder
json.extract! @item, :id, :name, :description, :picture, :documents, :created_at, :updated_at
```  
### base 64 upload
 Uploading base64-encoded files is a very easy task with Carrierwave. Just add the [carrierwave-base64](https://github.com/lebedev-yury/carrierwave-base64) gem:
 
```ruby
# Gemfile.rb
 gem 'carrierwave-base64'
```  
Install it:
```bash
bundle install
```  
 Go to your models and replace <code> mount_uploader </code> with <code>mount_base64_uploader </code>
```ruby
#app/models/item.rb
mount_base64_uploader :picture, PictureUploader
```   
```ruby  
#app/models/document.rb
mount_base64_uploader :document, DocumentUploader
```
And voila, your application can now accept base64-encoded files. It is as simple as that.
## Format of the request

### Using multipart form data
 For POSt-ing multipart form data, [cURL](https://curl.haxx.se/docs/manpage.html) is convenient enough. Keep in mind that the paths included are arbitrary and you have to add your own. Another thing - when sending files, make sure to declare their type (<code>type=application/pdf</code>). Otherwise, cURL sends files as a binary type (<code> application/octet-stream </code>); this may break some of the validations.
```bash
curl 
-F "item[document_data][]=@E:ile2.pdf;type=application/pdf" 
-F "item[document_data][]=@E:ile1.pdf;type=application/pdf"
-F "item[picture]=@E:photo.jpg" 
-F "item[name]=item" 
-F "item[description]=desc"  
localhost:3000/items
```
### Using base64 strings in JSON

 If you want to send base64 data, keep in mind that the strings are long, and cURL might be tedious to use. I would recommend [Postman](https://www.getpostman.com/docs/) . **Note: Replace "picture" with " image_base" if you used Paperclip**. Here is the JSON format that the API is going to accept:
 
 ```json
  {
   "item": {
        "name": "tem",
        "description": "desc",
        "picture": "data:image/png;base64, Rt2...",
        "document_data": [ "data:application/pdf;base64, fW3..." , "data:application/pdf;base64, pLf..."]
 
         }
  }
 
 ```
## Conclusion

Now you know how to upload files. You can now go ahead and set up a client-side application that can consume the API. In case you got lost in the guide, I created a [Github repository](https://github.com/hggeorgiev/rails-api-fileupload-tutorial) and added [branches](https://github.com/hggeorgiev/rails-api-fileupload-tutorial/branches/all) for each of the steps and the implementations done in the tutorial.

My personal preference for file upload is [CarrieWave](https://github.com/carrierwaveuploader/carrierwave) because it offers a more straightforward approach. The logic for the file is isolated into uploaders, which can be reused repeatedly for multiple models. This makes the code more maintinable. It took me considerably less time to implement file uploading for both single and multiple items. With the [CarrierWave-base64](https://github.com/lebedev-yury/carrierwave-base64) gem, I did not have to add any custom logic for decoding and saving images - changing the name of the mounting method was enough. **The only significant drawback is that CarrierWave has a lot of "magic" going behind it.** It does not feel as customizable as Paperclip, and debugging on CarrierWave seems more difficult.

On the other hand, [Paperclip](https://github.com/thoughtbot/paperclip) offers a more personalized approach to file uploads. All the logic is stored inside the model, and there is no need to encapsulate the logic outside the model. Files are treated more like standard fields in the model rather than independent parts that are merely associated with the model. Debugging is somewhat intuitive, since any errors are simply treated as attribute validation errors. On the other hand, making the image uploading more generic requires more tinkering and experience (putting methods in <code>ApplicationRecord</code> is one way). Moreover, Base64 uploads do not come as naturally as they do for CarrierWave, so there is a need for custom logic. 
 
To conclude, both CarrierWave and Paperclip have their pros and cons, and the choice of which to use depends on the case. Both gems are popular and have big communities built around them. The same applies for the different ways of uploading files - both base64 and multipart forms are widely-used means of uploading files to APIs. The choice depends on the scope and requirements of the project you are working on.

Please add any feedback you might have regarding the Ruby files upload process in the comments section. I'd be happy to answer any of your questions. Hope that you were piqued by this tutorial!
