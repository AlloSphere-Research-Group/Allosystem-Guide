# Writing applications using AlloSystem

This tutorial will focus on using the AlloSystem command line building facilities using the *run.sh* script. The code should work unchanged if you use AlloProject or your own IDE project, but the building procedure might differ to the instructions below.

##The "Steps"

Matt Wright's and Karl Yerkes' "Steps" are a series of C++ programs starting from "Hello, World!" and incrementally growing to an interactive (well, at least navigable) world of colored spheres each moving up and down to show the current pitch of the algorithmically generated sinewave "song" it is looping.  The diff from each to the next is always reasonably small and comprehendable, as we recommend in general as a best practice for incremental development and check-in of AlloSystem programs.

### Hello, World! in main() with std::cout

An AlloSystem program must start as any other C++ program with a main function:

    #include <iostream>

    int main(int argc, char *argv[])
    {
        std::cout << "Hello World!" << std::endl;
      return 0;
    }

If you save this text to a file called *allotutorial.cpp* in the *projects/* directory of AlloSystem, you can run it by typing on the shell (from the root AlloSystem source directory):

    ./run.sh projects/allotutorial.cpp

### Hello, World! in constructor of al::App

To make an "AlloApp", you need to write a class that inherits from the AlloSystem *App* class. This class is defined in the header *allocore/io/al_App.hpp*. We will use a *struct* instead of a class, to simplify inheritance and make all members public (i.e. no need to worry about what is public, private or protected). Note that this is not usually good practice as it breaks the encapsulation paradigm of object oriented programming, but it makes it easier to code simple applications.

    #include <iostream>
    #include <allocore/io/al_App.hpp>

    struct AlloApp: al::App {
      AlloApp() {
        std::cout << "Hello World!" << std::endl;
      }

    };

    int main(int argc, char *argv[])
    {
      MyApp app;
      return 0;
    }

Remember that you need to prefix the namespace *al* to all AlloSystem classes. An alternative is to bring the *al* namespace into the global namespace, which saves the need to prefix the class:

    #include "allocore/io/al_App.hpp" // gets us App
    using namespace al;
    using namespace std;

    struct AlloApp : App {
        AlloApp() {
            cout << "Created AlloApp" << endl;
        }
    };

    int main() {
        AlloApp app;
        return 0;
    }

Notice that in the last example we didn't use the arguments:

    (int argc, char* argv[])

in the main function. Command line arguments or flags are passed to your application through these variables. Command line arguments are used to adjust the way an application runs, for example, you can pass a filename, the resolution of your application, or any other type of global or initial configuration. If you don't need to use command line arguments in your application, you can leave these out.

### Open a window

AlloSystem "Apps" provide the facility to create windows. You can call the *initWindow()* function of the *app* object to initialize a window, and then the *start()* function to start the GUI processing loop, which will keep the program running until the window is closed.

    #include "allocore/io/al_App.hpp"
    using namespace al;
    struct AlloApp : App {
      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        initWindow();
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

### Make a sphere mesh and draw it in the window

To draw items on the screen, you must reimplement the *onDraw()* virtual function. Reimplementing a function means making a function in a child class (or struct) that replaces an exisitng one in the parent. What "virtual" means is that when the parent al::App class needs to draw items on the screen, it will call the function that you have defined, instead of its own original *onDraw()* function.

There is a default interface to the graphics rendering created with the parent *App*. It is called *g*, and you can draw a mesh in it by passing a *Mesh* object to its *draw()* function.

First, add a Mesh object called *m* to the *AlloApp* struct:

    struct AlloApp : App {
      Mesh m;

Then within we will reimplement *onDraw()* like this:

      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        g.draw(m);
      }
    };

Notice that the types for *g* and *v* have a *&* character at the end. This means they are passed *by reference*. When a variable is passed by reference, it is not copied before entering the new function, but it is the same actual values in memory that are passed, so any changes you make to the variable will affect the variable in the calller functions. This is similar to the use of pointers in C, but the difference is that you use the variable as a regular object rather than having to *dereference* the pointer.

Also notice that since the *m* Mesh is a member of the struct, we don't need to pass it as an argument. It is available to all the member functions of the struct.

An *AlloApp* has keyboard and mouse navigation active by default. Drag the mouse or use the arrow keys to look in a different direction. You can move the "camera" with the W, A, D and X keys. You can rotate the camera around its axis with Q and Z and if you want to move this rotation axis you can use E and C. The *Esc* key will toggle fullscreen and Ctrl+Q will quit the application.

If you navigate the scene, you will notice that we start in the center of the sphere!

###Use the OpenGL TRIANGLES primitive to make it look solid

In the *AlloApp* constructor, you can specify the type of primitives that the mesh will be rendered with:

    struct AlloApp : App {
      Mesh m;
      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        initWindow();
      }

      ...

    };

Now the sphere will be rendered using triangles, rather than points. Since we are in the middle of the sphere, you will see white no matter where you look!

###Initialize camera position to outside the sphere

To set the initial camera position, we need to call the *pos()* function to the *nav()* function.

    struct AlloApp : App {
      Mesh m;
      AlloApp() {
        nav().pos(0, 0, 10);

        addSphere(ourMesh);
        m.primitive(Graphics::TRIANGLES);
        initWindow();
      }
     ...
    };

The *pos()* function takes x, y and z coordinates. You can see that we set the z to 10, which moved the camera towards you 10 units. So the z axis is by default the depth of the scene.

###Try positioning a light too

You might have noticed that the sphere doesn't really look three-dimensional. This is because there are no lights in the scene reflecting from the sphere, to make it have volume.

To add a light to a scene, you first need to add it as a member to the struct, just like the mesh:

    struct AlloApp : App {
      Light light;
      Mesh m;

Then, you need to position the light in the scene:

      AlloApp() {
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);

Finally, you need to make sure the light is drawn every frame by calling the Light object as if it was a function:

      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        g.draw(m);
      }
    };

But wait! It still doesn't look right.

###You also have to compute normals to get lighting effects

To make sure the light in a scene does what it should, you need to generate the normals to the mesh, that way, light will know how it "reflects" off of each triangle.

      AlloApp() {
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.generateNormals();
        initWindow();


Voilà! Now the sphere appears to have volume.

###Raw sound: impulse train at the audio block rate

// - show doxygen on initAudio, onSound,
// - introduce the generator pattern: io() and AudioIOData
// - explain the waveform (pulse) and pitch of the sound
// - show adjusting audio buffer size with initAudio
//

    struct AlloApp : App {
      Light light;
      Mesh m;
      AlloApp() {

        ...

        initWindow(Window::Dim(200,300), "Title", 10);
        initAudio();
      }
      ...

      virtual void onSound(AudioIOData& io) {
        // this will make a pulse wave with a frequency of 44100 / 128 Hz
        io();
        io.out(0) = io.out(1) = 1.0f; // left = right = 1.0f;
        while (io()) {
          io.out(0) = io.out(1) = 0.0f;
        }
      }

    };


###Interactive sound

trigger one impulse per key press (via bool flag because threads)

// - introduce the type: bool
// - explain the concept of a flag
// - lookup onKeyDown, ViewpointWindow, and Keyboard in the doxygen
// - breifly mention threading. we'll return to it later
//

    #include "allocore/io/al_App.hpp"
    using namespace al;
    struct AlloApp : App {

      bool shouldClick;
      Light light;
      Mesh m;

      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.generateNormals();
        initWindow();
        initAudio();

        shouldClick = true;
      }

      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        g.draw(m);
      }

      virtual void onSound(AudioIOData& io) {

        while ( io() ) {

          if (shouldClick) {
            shouldClick = false;

            float s = 1.0f;
            io.out(0) = s;
            io.out(1) = s;

          }
          else {
            float s = 0.0f;
            io.out(0) = s;
            io.out(1) = s;
          }
        }
      }

      virtual void onKeyDown(const ViewpointWindow& vw, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
          shouldClick = true;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Play a sine wave
(using a gam::sine<> unit generator)

// - show Gamma doxygen on Sine
// - explain "gam::Sync::master().spu(audioIO().fps());"
// - use vocabulary: templates, generator, frequency, phase, samplerate
//
    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    struct AlloApp : App {
      Light light;
      Mesh m;

      gam::Sine<> sine;

      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.generateNormals();

        sine.freq(1400.0);

        initWindow();
        initAudio();
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        g.draw(m);
      }

      virtual void onSound(AudioIOData& io) {

        // tell gamma's oscillators what the sample rate is
        gam::Sync::master().spu(audioIO().fps());

        while (io()) {
          float s = sine();
          io.out(0) = s;
          io.out(1) = s;
        }

      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }



###Draw two spheres
(the same mesh translated)

// - show doxygen on Graphics
// - explain pushMatrix(), popMatrix(), and translate()
//

    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    struct AlloApp : App {
      Light light;
      Mesh m;
      gam::Sine<> sine;

      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);

        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.generateNormals();

        sine.freq(440.0);

        initWindow();
        initAudio();
      }

      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        g.draw(m);

        g.translate(2, 1, 0);
        g.draw(m);
      }

      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());
        while (io())
          io.out(0) = io.out(1) = sine();
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }



###Play two sines
(each has its own phase so you need two instances of the unit generator)

// - does this scale?
// - how do we make lots of spheres and sines?
//

    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    struct AlloApp : App {
      Light light;
      Mesh m;

      gam::Sine<> sine, sine2;

      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.generateNormals();

        sine.freq(440.0);
        sine2.freq(770.0);

        initWindow();
        initAudio();
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        g.draw(m);

        g.translate(2, 1, 0);
        g.draw(m);
      }
      virtual void onSound(AudioIOData& io) {

        gam::Sync::master().spu(audioIO().fps());

        while (io()) {
          io.out(0) = sine(); // left
          io.out(1) = sine2(); // right
          //io.out(0) = io.out(1) = 0.5f * sine() + 0.5f * sine2();
          //io.out(0) = io.out(1) = (sine() + sine2()) * 0.5f;
        }
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }



###Play 20 sines

(an array of ugens, for loop to set frequencies)

// - explain for, #define, arrays and the / N
//
    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;

    #define N 20

    struct AlloApp : App {
      Light light;
      Mesh m;

      gam::Sine<> sine[N]; // array [] .. the way to make many of something

      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.generateNormals();

        for (int i = 0; i < N; ++i)
          sine[i].freq(220.0 + i * i); // 220, 221, 224, ... 581

        initWindow();
        initAudio();
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        g.draw(m);
        g.pushMatrix();
        g.translate(2, 1, 0);
        g.draw(m);
        g.popMatrix();
      }



      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());

        while (io()) {
          float s = 0;
          for (int i = 0; i < N; ++i)
            s += sine[i]();
          io.out(0) = io.out(1) = s / N;
        }
      }

      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Draw 20 spheres
(an array of 3D positions, initialized with for loop, push/pop OpenGL matrices)

    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    #define N 20
    struct AlloApp : App {
      Light light;
      Mesh m;
      gam::Sine<> sine[N];

      Vec3f position[N];

      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.generateNormals();

        for (int i = 0; i < N; ++i) {
          sine[i].freq(220.0 + i * i);
          position[i] = Vec3f(sin(i), i, -i * i);
        }

        initWindow();
        initAudio();
      }

      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();

        for (int i = 0; i < N; ++i) {
          g.pushMatrix();
          g.translate(position[i]);
          g.draw(m);
          g.popMatrix();
        }
      }
      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());
        while (io()) {
          float s = 0;
          for (int i = 0; i < N; ++i)
            s += sine[i]();
          io.out(0) = io.out(1) = s / N;
        }
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Scaling the one sphere mesh affects all 20 times its drawn

// - explain why scale() effects all spheres
//
    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    #define N 20
    struct AlloApp : App {
      Light light;
      Mesh m;
      gam::Sine<> sine[N];
      Vec3f position[N];
      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.scale(0.2);
        m.generateNormals();
        for (int i = 0; i < N; ++i) {
          sine[i].freq(220.0 + i * i);
          position[i] = Vec3f(sin(i), i, -i * i / 7);
        }
        initWindow();
        initAudio();
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        for (int i = 0; i < N; ++i) {
          g.pushMatrix();
          g.translate(position[i]);
          g.draw(m);
          g.popMatrix();
        }
      }
      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());
        while (io()) {
          float s = 0;
          for (int i = 0; i < N; ++i)
            s += sine[i]();
          io.out(0) = io.out(1) = s / N;
        }
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Animate sphere positions with random walk

// - explain why we use onAnimate() rather than onDraw()
// - mention Vec += operator
//

    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    #define N 20
    struct AlloApp : App {
      Light light;
      Mesh m;
      gam::Sine<> sine[N];
      Vec3f position[N];
      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.scale(0.2);
        m.generateNormals();
        for (int i = 0; i < N; ++i) {
          sine[i].freq(220.0 + i * i);
          position[i] = Vec3f(sin(i), i, -i * i / 7);
        }
        initWindow();
        initAudio();
      }
      virtual void onAnimate(double dt) {
        for (int i = 0; i < N; ++i)
          position[i] += Vec3f(rnd::uniformS(), rnd::uniformS(), rnd::uniformS()) * 0.01;
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        for (int i = 0; i < N; ++i) {
          g.pushMatrix();
          g.translate(position[i]);
          g.draw(m);
          g.popMatrix();
        }
      }
      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());
        while (io()) {
          float s = 0;
          for (int i = 0; i < N; ++i)
            s += sine[i]();
          io.out(0) = io.out(1) = s / N;
        }
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Make each sine’s frequency depend on corresponding sphere’s position

Shoudn't there be thread protection?

    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    #define N 20
    struct AlloApp : App {
      Light light;
      Mesh m;
      gam::Sine<> sine[N];
      Vec3f position[N];
      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.scale(0.2);
        m.generateNormals();
        for (int i = 0; i < N; ++i) {
          sine[i].freq(220.0 + i * i);
          position[i] = Vec3f(sin(i), i, -i * i / 7);
        }
        initWindow();
        initAudio();
      }
      virtual void onAnimate(double dt) {
        for (int i = 0; i < N; ++i) {
          position[i] += Vec3f(rnd::uniformS(), rnd::uniformS(), rnd::uniformS()) * 0.01;
          sine[i].freq(220.0 + position[i].mag() * 5);
        }
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        for (int i = 0; i < N; ++i) {
          g.pushMatrix();
          g.translate(position[i]);
          g.draw(m);
          g.popMatrix();
        }
      }
      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());
        while (io()) {
          float s = 0;
          for (int i = 0; i < N; ++i)
            s += sine[i]();
          io.out(0) = io.out(1) = s / N;
        }
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Map sine amplitude to the inverse of view distance

Again threading issues?

    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    #define N 20
    struct AlloApp : App {
      Light light;
      Mesh m;
      gam::Sine<> sine[N];
      Vec3f position[N];
      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.scale(0.2);
        m.generateNormals();
        for (int i = 0; i < N; ++i) {
          sine[i].freq(220.0 + i * i);
          position[i] = Vec3f(sin(i), i, -i * i / 7);
        }
        initWindow();
        initAudio();
      }
      virtual void onAnimate(double dt) {
        for (int i = 0; i < N; ++i) {
          position[i] += Vec3f(rnd::uniformS(), rnd::uniformS(), rnd::uniformS()) * 0.01;
          sine[i].freq(220.0 + position[i].mag() * 5);
        }
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        light();
        for (int i = 0; i < N; ++i) {
          g.pushMatrix();
          g.translate(position[i]);
          g.draw(m);
          g.popMatrix();
        }
      }
      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());
        while (io()) {
          float s = 0;
          for (int i = 0; i < N; ++i)
            s += sine[i]() / (nav().pos() - position[i]).mag();
          io.out(0) = io.out(1) = s / N;
        }
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Clip range of possible amplitudes so no sine is too loud nor too quiet.   Add color to spheres.

// - explain why Material is necessary
// - explain HSV
//
    #include "allocore/io/al_App.hpp"
    #include "Gamma/Oscillator.h"
    using namespace al;
    #define N 20
    struct AlloApp : App {
      Material material;
      Light light;
      Mesh m;
      gam::Sine<> sine[N];
      Vec3f position[N];
      AlloApp() {
        std::cout << "Created AlloApp" << std::endl;
        nav().pos(0, 0, 10);
        light.pos(0, 0, 10);
        addSphere(m);
        m.primitive(Graphics::TRIANGLES);
        m.scale(0.2);
        m.generateNormals();
        for (int i = 0; i < N; ++i) {
          sine[i].freq(220.0 + i * i);
          position[i] = Vec3f(sin(i), i, -i * i / 7);
        }
        initWindow();
        initAudio();
      }
      virtual void onAnimate(double dt) {
        for (int i = 0; i < N; ++i) {
          position[i] += Vec3f(rnd::uniformS(), rnd::uniformS(), rnd::uniformS()) * 0.01;
          sine[i].freq(220.0 + position[i].mag() * 5);
        }
      }
      virtual void onDraw(Graphics& g, const Viewpoint& v) {
        material();
        light();
        for (int i = 0; i < N; ++i) {
          g.pushMatrix();
          g.color(HSV((float)i / N, 1, 1));
          g.translate(position[i]);
          g.draw(m);
          g.popMatrix();
        }
      }

      virtual void onSound(AudioIOData& io) {
        gam::Sync::master().spu(audioIO().fps());
        while (io()) {
          float s = 0;
          for (int i = 0; i < N; ++i)
            s += sine[i]() / (nav().pos() - position[i]).mag();
          io.out(0) = io.out(1) = s / N;
        }
      }
      virtual void onKeyDown(const ViewpointWindow&, const Keyboard& k) {
        if (k.key() == ' ') {
          std::cout << "Spacebar pressed" << std::endl;
        }
      }
    };
    int main(int argc, char* argv[]) {
      AlloApp app;
      app.start();
      return 0;
    }

###Refactoring:  making an Agent data structure (sine ugen plus sphere’s 3D position)


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
