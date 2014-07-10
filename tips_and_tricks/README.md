# Tips and Tricks

Below you will find information on techniques commonly needed for interactive applications.

##Using OSC

AlloSystem provides in the Allocore library facilities to use OSC (Open Sound Control).


##Threading and audio

Audio processing runs as a separate high priority thread, calling the *onSound()* function asynchronously. In other words, the rhythm at which this audio function is called is not synchronized to graphics or input events. This can cause unpredictable crashes if two threads try to access the same memory location simultaneously. In most applications, a mutex (mutually exclusive lock) is generally used to protect data. However, it's not recommended to use mutexes within the audio callback, as they can block the function long enough to miss the tight deadlines, causing audio dropouts.

A simple way to stream values to and from the audio callback is to use a lock-free ring-buffer. To use it first

    #include "allocore/system/al_SingleRWRingBuffer.hpp



##Broadcast App
