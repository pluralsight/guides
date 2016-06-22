Last year I needed to build a WPF application for an electronic stethoscope which is able to record respiratory audio, save them to wave files, and if user wants, play the wave files at a later time. At this point my only experience with audio in general was with Unity3D -which has some great tools for it- and Matlab back in my university days. I remember thinking to myself "How hard can it be? I already know C# can play wave files, there must be some advanced tools in the core libraries!". I was, ofcourse, very *very* wrong.

Not only I was shocked to find no core libraries, I was also amazed how audio is a very deep and challenging subject in programming. Lucky for me, Pluralsight has great courses on [audio in general](https://www.pluralsight.com/courses/digital-audio-fundamentals) and on [NAudio](https://www.pluralsight.com/courses/audio-programming-naudio), so I was able to finish my project successfully.

I believe to learn something properly, you need to create a basic project using it. So in this tutorial I will explain how to build a simple media player from ground up using a very popular .NET audio library NAudio.

### Why Build Your Own Media Player?
Other than the fact that it is fun work on your own pet projects:
- You might have a project where you need to make a in-app media player to achieve your goals like my situation.
- You might need a very special feature that no other media player provides.
- You might also not trust 3rd party media players in your secure environment where you absolutely have to use a media player.

Whatever your reason is, you have challenges waiting for you down the road when it comes to audio programming.

### Challenges
The first challenge is the concept that audio is not a traditional object, but a stream object. A stream is usually a sequence of bytes. If you want to read it, it has to be read like an array. Also it can't be manipulated easily, you have to dig into the byte array and figure out what to do with each byte to achive what you want. They are not very easy to debug either. There are also considerations when it comes to audio such as, number of channels, frequency, file formats etc. Especially when it comes to debugging it can be a mess.

There are questions need answering such as "How do we know if we reach the end of an audio file?", "How do we stop or pause the stream?", "How do we resume where we left off?". Ofcourse there are many more issues you need to address but these are the core issues.

Finally there are memory management issues. Typically, when you are done with a stream you have to dispose of it properly otherwise it will stay in the memory causing a memory leak. For example, if you are working with hundereds of audio and you don't dispose them properly, you could run out of memory really fast and your application may crash. So in essence, you have to manually manage memory.

So when you think about audio considering these challenges, even a simple action such as playing an audio file, gets very complicated very fast.

### NAudio to the Rescue
Lucky for us, there is an audio library for .NET - NAudio - that will do most of this work for us. We will still be working with streams and we will have to face several challanges that comes with them, however NAudio is a great library that abstracts us from the details of simple playing and recording of audio files. The good thing about NAudio libarary is, it can be used in every type of project. If you simply want build and application to play or record audio files, you can do that without delving deep into streams. If you want something complex such as manipulating audio or creating filters it provides very good tools for that too.

There are 3 ways of getting NAudio:
- [NAudio codeplex page](https://naudio.codeplex.com/)
- [NAudio GitHub page](https://github.com/naudio/NAudio)
- And finally via [NuGet](https://www.nuget.org/packages/NAudio/)

I usually go with NuGet since it is the easiest of them all, however if you want to see how it actually works there are great examples and documentation on the Codeplex and GitHub pages.

### Media Player

The main purpose of this tutorial is to build a simple media player which will have some common features of various media players such as:
- Play audio files with various formats (wav, mp3, ogg, flac etc)
- Skip to beginning, skip to end, play/pause, stop, shuffle buttons and their functionality
- Have a volume control
- Have a seekbar
- Have a playlist
- Add a file to this playlist
- Add a folder of files to this playlist
- Save and load playlists

It should look like this at the end (with my Witcher 3 soundtrack playlist):
![Our finalized media player](https://raw.githubusercontent.com/pluralsight/guides/master/images/71e4ccf5-4301-4b3f-8cb2-7fde9b7a816e.png)

### Implementation
In the solution, we will have two projects. One for the actual application, and one for the NAudio abstraction. NAudio is great in abstracting us from details but I want to make it very easy for us to do everything with a few methods, rather than calling NAudio codes for playing, pausing, stopping etc.

However there is a choice we have to make when it comes to the main project where the UI is; do we use the traditional event driven architecture or do we use **M**odel **V**iew **V**iew **M**odel architecture (MVVM). You might think that you would use MVVM because that's all the cool kids use nowadays, however both architectures have advantages and disadvantages especially when it comes to developing a real time application such as this. I developed media players using both architectures on two different projects:
- If you use MVVM you write way less code, however in this case, property binding has a nasty habit of breaking down and causing bugs if you bind the wrong way. Especially when it comes to implementing a real time changing two-way bound seekbar control.
- If you use the good old event driven architecture you get to write a lot more UI event code, however you will have full control over what happens during those events so it is easier to implement a real time control such as a seekbar.

Since this is a brand new project I would go with MVVM, so in this tutorial I will use the MVVM architecture. However since this is not an MVVM tutorial I won't go into details on how MVVM works.

#### The UI
I always like to start with the UI part when it comes to WPF projects because it provides me with the features I need to implement to code in a visual way. So I will start coding by first generating a UI mock-up so we can see what features we need.

Before we start on the UI however, I will provide you the link for the images I used for the buttons so you will have them ready. For icons I used Google's free material design icons. You can get them from Google's material design [page](https://material.google.com/resources/sticker-sheets-icons.html#). When you download the icons, do a quick search of "play" or "pause" in the download directory to find the relevant icons.

Here is our XAML code for the UI:
```xaml
```















