Sending basic JSON data made out of strings to an API is, in most cases, a simple and straightforward task. But what about sending files, which are made of numerous lines of binary data and consits of various formats? Such data requires a slightly different approach and there are several formats through which files can be sent to the API.

In this guide, the two main approaches of handling file upload, base64 encoding and multipart form data, will be examined. The approaches will be implemented in a Rails 5 API application using both the [paperclip](https://github.com/thoughtbot/paperclip) and the [carrierwave](https://github.com/carrierwaveuploader/carrierwave) gems. A sample AngularJS application will also be written in order to present how the uploads can be handled client-side.

# Approaches for sending files to a Rails 5 API 
## Multipart form data
  Multipart forms have been around since [HTML 4](https://www.w3.org/TR/html401/interact/forms.html#h-17.13.4). They were introduced because the standard *application/x-www-form-urlencoded* forms did not handle bigger amounts of data well enough. According to [W3C](https://www.w3.org/)'s definition of *multipart/form-data*, it is used for forms that "contain files, non-ASCII data, and binary data". What multipart forms do differently is that they break the data in different chunks that represent the various charactereistics of the file (size, content type, name, contents) as well as the characteristics of the standard text and number data that is also sent with the form.
  
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
If the user enters "John" in the text input, and selects the text file "file1.txt", the user agent might send back the following data:

```
   Content-Type: multipart/form-data; boundary=AaB03x

   --AaB03x
   Content-Disposition: form-data; name="submit-name"

   Larry
   --AaB03x
   Content-Disposition: form-data; name="files"; filename="file1.txt"
   Content-Type: text/plain

   ... contents of file1.txt ...
   --AaB03x--

```
Every part of the form is separated by a boundary, which reprsents a different string (<code>AaB03x</code> in the example). Each parts contains information about the <code>Content-Type</code> it contains and the content of the part itself.

Let's consider uploading another file. If the user selects another file  "file2.gif", the browser will construct the parts as follows:
```
   Content-Type: multipart/form-data; boundary=AaB03x

   --AaB03x
   Content-Disposition: form-data; name="submit-name"

   Larry
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

Here, it can be seen that there is another part added to the <code>form-data</code>. This time, since the file is not in a <code>text/plain</code> format, it is broken down in binary, as it can be inferred from the <code> Content-Transfer-Encoding</code> property. The <code>Content-Type</code> property gives information about the type of the file, also known as its [media (MIME) type](https://en.wikipedia.org/wiki/Media_type). If the <code> Content-Type </code> property  is not defined and  the file is not in text format, its format will default to <code> application/octet-stream </code> which means that the finaly is binary and has no type. When sending data to an API, it is always good to include a <code>Content-Type</code> to each part which contains a file, otherwise there would be no way to validate the contents of the file.
## Base64 encoding
