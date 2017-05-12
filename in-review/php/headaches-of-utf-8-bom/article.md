I love Windows, but not when developing PHP applications. Recently, I have started working on too many main-stream PHP and NodeJS applications and REST APIs. Considering the REST APIs, the output must be perfect. Even a single character space will screw the 💩 out of the system. 😩

It is therefore, due to this fact, when designing API end-points, we must make sure that the output doesn’t contain any encoding marks or unnecessary characters. Also, we should be aware that these invisible buggers will not get generated because of user intervention, but due to the way, the file gets saved to the file system. 🤦🏻‍♂️

## Problem

The real problem lies in the way the text editors save the file to the file system. Generally, in the case of Windows, the text editors save either in UTF-8 (with BOM, without BOM), UTF-16 (with BOM, without BOM, little endian, etc.), ANSI, or Windows-1252. The worst that happens is, when every file is saved, it gets a Byte-Order-Mark or BOM appended at the end of file. 😓

This will pose a severe threat while developing API end-points, where a single UTF-8 BOM might make the JSON / HTTP Response totally invalid. My current system (not my permanent or personal) is a Windows machine, with Vagrant (+VirtualBox) installed in it. I am writing the files on the host (Windows) machine, which automatically saves along with the BOM.

I found the issue by checking the response from the server to be unusual. Both the images are from Chrome Developer Tools, where the first image is from Inspecting the element and the second one is the response from the server.

![Before-InspectElement](https://i.imgur.com/zKqEbsm.png)

![Before-NetworkResponse](https://i.imgur.com/DI9ZZF1.png)
 
The UTF-8 BOM characters can be identifiable using three hexadecimal character signature specified by `EF BB BF`.

## Solution

The only solution to avoid this is to get rid of the UTF-8 BOM completely, by using two simple commands (Mac or Linux). 😁 The first command is to find the files that have the BOM and the second one is to remove (or replace with empty string) the BOM.

    grep -rl $'\xEF\xBB\xBF' .

The second command that helps to remove is using `sed`. The command is as follows:

    sed -i '1 s/^\xef\xbb\xbf//' *.php

You may replace the `*.php` with anything like `*.inc` or a single file `index.php`. It is a pain if the files are inside multiple sub-folders. Hope this solved someone’s time in getting it out. Once the above has been done, I could see that the unusual characters no longer appear in the output. 😁

![After-InspectElement](https://i.imgur.com/TNyWwzc.png)

![After-NetworkResponse](https://i.imgur.com/mEgRKtW.png)
