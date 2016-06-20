Last year I needed to build a WPF application for an electronic stethoscope which is able to record respiratory audio, save them to wave files, and if user wants, play the wave files at a later time. At this point my only experience with audio in general was with Unity3D -which has some great tools for it- and Matlab back in my university days. I remember thinking to myself "How hard can it be? I already know C# can play wave files, there must be some advanced tools in the core libraries!". I was, ofcourse, very *very* wrong.

Not only I was shocked to find no core libraries, I was also amazed how audio is a very deep and challenging subject in programming. Lucky for me, Pluralsight has great courses on [audio in general](https://www.pluralsight.com/courses/digital-audio-fundamentals) and on [NAudio](https://www.pluralsight.com/courses/audio-programming-naudio), so I was able to finish my project successfully.

I believe to learn something properly, you need to create something basic using it. So in this tutorial I will explain how audio works on a programming level and how to build a simple media player from ground up using a very popular .NET audio library NAudio.

### Welcome to the World of Streams
My first challenge was to wrap my head around the concept that audio is not an object, but a stream object. A stream is usually a sequence of bytes. If you want to read it, it has to be read like an array. Also it cannot be manipulated easily, you have to dig into the byte array and figure out what to do with each byte to achive what you want. There are also considerations when it comes to audio such as, number of channels, frequency, file formats etc.

It doesn't stop there. When you are done with a stream you have to dispose of it properly otherwise it will stay in the memory causing a memory leak. For example, if you are working with hundereds of audio you could run out of memory really fast and your application may crash. So in essence, you have to manually manage memory.

So when you think about audio this way, even just playing an audio file gets very complicated very fast.

### NAudio to the Rescue
Lucky for us, there is a audio library for .NET called NAudio that will do most of this work for us. We will still be working with streams; there is no avoiding that when it comes to audio, however NAudio is a great library that abstracts us from the details of simple playing and recording of audio files. The good thing about NAudio is, it can be used in every type of project. If you simply want build and application to play or record audio files, you can do that without delving deep into streams. If you want something complex such as manipulating audio or creating filters it provides very good tools for that too.

There are 3 ways of getting NAudio:
- [NAudio codeplex page](https://naudio.codeplex.com/)
- [NAudio GitHub page](https://github.com/naudio/NAudio)
- And finally via [NuGet](https://www.nuget.org/packages/NAudio/)

I usually go with NuGet since it is the easiest of them all, however if you want to see how it actually works there are great examples and documentation on the Codeplex and GitHub pages.

### Media Player

The main purpose of this tutorial is to build a simple media player which will have some common features of various media players such as:
- Play audio files with various formats (wave, mp3, ogg, flac etc)
- Have a playlist
- Add files to this playlist
- Add folders of files to this playlist








