I recently set out to create a meta build system for Common Lisp projects, and ran into a problem I didn't expect: fetching the dependencies for a project can get complicated quickly.

First, some background. When building an executable out of a lisp system, there is usually only one approach available: dumping the running lisp image to disk. This means that whatever code is loaded during the build process gets included in the final executable. This isn't ideal, as it means that the build system infrastructure gets included in the final image. If you download your system's dependencies and build your image within the same step, it means that the ability to load arbitrary code from the Internet also gets included in your system! This may be useful for projects that expose a REPL to users, but it also means that if you pulled in your changes with ocicl but you users want to use qlot to manage their own dependencies, they can't.

Therefore, one of the goals of this build system is to completely separate building a CL project into 2 distinct steps:

1. Download all of the systems the project depends on.
2. Build the system.

This allows use to not include build infrastructure we don't need within the final image*.

Having these two steps be separate sounds pretty normal, but a usual, CL's flexibly complicates things. Due to having the loading commands available at runtime, it turns out that making #2 and #3 completely distinct is impossible; the only way to known all of the dependencies for a system is to build it. Existing project managers solve this in one of two ways; quicklisp (and therefore the managers that use its data) build everything beforehand and keep a record of what was loaded during the build. This data is then used to fetch all required dependencies at once before the code is compiled locally. The second option is to use the same mechanism to hook into the build system, but fetch the code on demand so they don't need to prebuild the systems that they ship.

Since loading code outside of the build system is an insane thing to do, we can safely ban that from being done in our target system. However, we still need to support it for the projects' dependencies, particularly the ones that are locally vendored and won't have dependency information included with our system manager. Therefore, the actual build process will involve three steps:

1. Download or find all of the project's dependencies that we know about.
2. Build the locally vendored dependencies with automatic fetching turned on.
3. Compile our project's code and dump the resulting image without the project manager code loaded.