# Writing applications using AlloSystem
The current AlloSystem Tutorial is the "Steps."

##The "Steps"

Matt Wright's and Karl Yerkes' "Steps" are a series of C++ programs starting from "Hello, World!" and incrementally growing to an interactive (well, at least navigable) world of colored spheres each moving up and down to show the current pitch of the algorithmically generated sinewave "song" it is looping.  The diff from each to the next is always reasonably small and comprehendable, as we recommend in general as a best practice for incremental development and check-in of AlloSystem programs.


The "Steps":
0. Hello, World! in main() with std::cout
1. Hello, World! in constructor of al::App
2. Have it open a window (with no graphics in it)
3. Make a sphere mesh and draw it in the window (as points).   Using default QWERTY navigation bindings.
4. Use the OpenGL TRIANGLES primitive to make it look solid
5. Initialize camera position to outside the sphere
6. Try positioning a light too (but it looks exactly the same?)
7. You also have to compute normals to get lighting effects
8. Raw sound: impulse train at the audio block rate
9. Interactive sound: trigger one impulse per key press (via bool flag because threads)
10. Play a sine wave (using a gam::sine<> unit generator)
11. Draw two spheres (the same mesh translated)
12. Play two sines (each has its own phase so you need two instances of the unit generator)
13. Play 20 sines (an array of ugens, for loop to set frequencies)
14. Draw 20 spheres (an array of 3D positions, initialized with for loop, push/pop OpenGL matrices)
15. Scaling the one sphere mesh affects all 20 times its drawn
16. Animate sphere positions with random walk
17. Make each sine’s frequency depend on corresponding sphere’s position
18. Map sine amplitude to the inverse of view distance
19. Clip range of possible amplitudes so no sine is too loud nor too quiet.   Add color to spheres.
20. Refactoring:  making an Agent data structure (sine ugen plus sphere’s 3D position)
21. Refactoring: Putting behavior (methods) into the agent class
22. Give each agent a “song” (algorithmically generated melodic pitch sequence)
23. Add amplitude envelopes
24. Synchronize graphics and audio: move agent’s sphere (randomly) only when pitch changes
25. Synchronize graphics and audio: animate Y-position of agent’s sphere to match its current pitch
26. Add glissandi (frequency envelopes) to certain notes of each melody
27. Make visual motion of balls follow glissandi


% thank you very much whoever turns this into a real table!

##Getting the Steps

mat201b2013 repo wOOt!



##AlloSystem Examples

AlloSystem includes examples organized by the same core/util + module directory structure as the header files.   So the graphics-related AlloCore examples are in

    allocore/examples/graphics

while the full-surround examples (which happen to be in AlloUtil) are in

    alloutil/examples

while the examples of how to link and coordinate MySQL with AlloSystem as a whole is in

    examples/mysql

Examples could use a lot more commenting, etc., to be more helpful as a "Tutorial".
