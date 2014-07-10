# Tips and Tricks

Below you will find information on techniques commonly needed for interactive applications.

##Using OSC

AlloSystem provides in the Allocore library facilities to use OSC (Open Sound Control).


##Threading and audio

Audio processes runs as a separate high priority thread, calling the *onSound()* function asynchronously. In other words, the rhythm at which this audio function is called is not synchronized to graphics or input events. This can cause unpredictable crashes if two threads try to access the same memory location simultaneously.



##Broadcast App
