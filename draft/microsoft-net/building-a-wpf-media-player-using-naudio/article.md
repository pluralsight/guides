Last year I needed to build a WPF application for an electronic stethoscope which is able to record respiratory audio, save them to wave files, and if user wants, play the wave files at a later time. At this point my only experience with audio in general was with Unity3D -which has some great tools for it- and Matlab back in my university days. I remember thinking to myself "How hard can it be? I already know C# can play wave files, there must be some advanced tools in the core libraries!". I was, ofcourse, very *very* wrong.

Not only I was shocked to find no core libraries, I was also amazed how audio is a very deep and challenging subject in programming. Lucky for me, Pluralsight has great courses on [audio in general](https://www.pluralsight.com/courses/digital-audio-fundamentals) and on [NAudio](https://www.pluralsight.com/courses/audio-programming-naudio), so I was able to finish my project successfully.

I believe to learn something properly, you need to create a basic project using it. So in this tutorial I will explain how to build a simple media player from ground up using a very popular .NET audio library NAudio.

### Challenges
The first challenge is the concept that audio is not an object, but a stream object. A stream is usually a sequence of bytes. If you want to read it, it has to be read like an array. Also it can't be manipulated easily, you have to dig into the byte array and figure out what to do with each byte to achive what you want. They are not very easy to debug either. There are also considerations when it comes to audio such as, number of channels, frequency, file formats etc. Especially when it comes to debugging it can be a mess.

There are questions need answering such as "How do we know if we reach the end of an audio file?", "How do we stop or pause the stream?", "How do we resume where we left off?". Ofcourse there are many more issues you need to address but these are the core issues.

Finally there are memory management issues. Typically, when you are done with a stream you have to dispose of it properly otherwise it will stay in the memory causing a memory leak. For example, if you are working with hundereds of audio and you don't dispose them properly, you could run out of memory really fast and your application may crash. So in essence, you have to manually manage memory.

So when you think about audio considering these challenges, even a simple action such as playing an audio file, gets very complicated very fast.

### NAudio to the Rescue
Lucky for us, there is an audio library for .NET -NAudio- that will do most of this work for us. We will still be working with streams and we will have to face several challanges that comes with them, however NAudio is a great library that abstracts us from the details of simple playing and recording of audio files. The good thing about NAudio libarary is, it can be used in every type of project. If you simply want build and application to play or record audio files, you can do that without delving deep into streams. If you want something complex such as manipulating audio or creating filters it provides very good tools for that too.

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











