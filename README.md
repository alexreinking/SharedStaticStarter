# SharedStaticStarter

![Test workflow](https://github.com/alexreinking/SharedStaticStarter/actions/workflows/test.yml/badge.svg)

A simple starter project showing how to distribute both static and shared libraries in CMake.

The basic challenge is that CMake targets represent a single physical library, so there are a bunch of
competing standards for determining whether `SomeLib::SomeLib` ought to be shared or static. In the
associated blog post, I develop this solution from three basic principles:

1. The build interface should match the install interface. 
2. Only strict build requirements belong in CMakeLists.txt.
3. A single project will not mix both shared and static versions of a library. 

For the full story, click here: https://alexreinking.com/blog/building-a-dual-shared-and-static-library-with-cmake.html

---

If this example helps you with your work, consider saying thank you by buying me a coffee!

[![Buy me a coffee](https://img.buymeacoffee.com/button-api/?text=Buy%20me%20a%20coffee&emoji=&slug=alexreinking&button_colour=40DCA5&font_colour=ffffff&font_family=Lato&outline_colour=000000&coffee_colour=FFDD00)](https://www.buymeacoffee.com/alexreinking)
