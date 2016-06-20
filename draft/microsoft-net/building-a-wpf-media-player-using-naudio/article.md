Last year I needed to build a WPF application for an electronic stethoscope which is able to record respiratory audio, save them to wave files, and if user wants, play the wave files at a later time. At this point my only experience with audio in general was with Unity3D -which has some great tools for it- and Matlab back in my university days. I remember thinking to myself "How hard can it be? I already know C# can play wave files, there must be some advanced tools in the core libraries!". I was, ofcourse, very *very* wrong.

Not only I was shocked to find no core libraries, I was also amazed how audio is a very deep and challenging subject in programming. Lucky for me, Pluralsight has great courses on [audio in general](https://www.pluralsight.com/courses/digital-audio-fundamentals) and on [NAudio](https://www.pluralsight.com/courses/audio-programming-naudio), so I was able to finish my project successfully.

I believe to learn something properly, you need to create something basic using it. So in this tutorial I will explain how audio works on a programming level and how to build a simple media player from ground up using a very popular .NET audio library NAudio.

#### Welcome to the World of Streams!
My first challenge was to wrap my head around the concept that audio is not an object, but a stream. A stream is usually a sequence of bytes. If you want to read it, it has to be read like an array. Also it cannot be manipulated easily, you have to dig into the byte array and figure out what to do with each byte to achive what you want. There are also considerations when it comes to audio such as, number of channels, frequency, file formats etc. So when you think about audio this way, even just playing an audio file gets very complicated very fast.

It doesn't stop there. When you are done with a stream you have to dispose of it properly otherwise it will stay in the memory causing a memory leak. For example, if you are working with hundereds of audio you could run out of memory really fast and your application may crash. So in essence, you have to manually manage memory.

#### NAudio to the Rescue
Lucky for us, there is a audio library called NAudio that will do most of this work for us. We will still be working with streams there is no avoiding that when it comes to audio, NAudio is a great library that abstracts us from the details of simple playing and recording of audio files. The good thing about NAudio is, it is for everyone. If you simply want to play an audio, you can do that without delving deep into streams. If you want something complex such as manipulating audio or creating filters, it provides very good tools for that too.








