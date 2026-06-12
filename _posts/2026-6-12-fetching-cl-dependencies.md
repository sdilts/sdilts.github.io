I recently set out to create a meta build system for Common Lisp projects, and ran into a problem I didn't expect: fetching the dependencies for a project can get complicated quickly.

First, some background. When building an executable out of a lisp system, there is usually only one approach available: dumping the running lisp image to disk. This isn't ideal, as it means that the build system infrastructure gets included in the final executable. If you download your system's dependencies and build your image within the same step, it means that the ability to load arbitrary code from the Internet also gets included in your system! This may be useful for projects that expose a REPL to users, but it also means that if you pulled in your changes with ocicl but you users want to use qlot to manage their own dependencies, they can't.

Therefore, one of the goals of this build system is to completely separate building a CL project into 2 distinct steps:

1. Download all of the systems the project depends on.
2. Build the system.

This allows use to not include build infrastructure we don't need within the final image[^final-image].

Having these two steps be separate sounds pretty normal, but a usual, CL's flexibly complicates things. Due to having the loading commands available at runtime, it turns out that making #2 and #3 completely distinct is impossible; the only way to known all of the dependencies for a system is to build it. Existing project managers solve this in one of two ways; quicklisp (and therefore the managers that use its data) build everything beforehand and keep a record of what was loaded during the build. This data is then used to fetch all required dependencies at once before the code is compiled locally. The second option is to use the same mechanism to hook into the build system, but fetch the code on demand so they don't need to prebuild the systems that they ship.

Since loading code outside of the build system is an insane thing to do, we can safely ban that from being done in our target system. However, we still need to support it for the projects' dependencies, particularly the ones that are locally vendored and won't have dependency information included with our system manager. Therefore, the actual build process will involve three steps:

1. Download or find all of the project's dependencies that we know about.
2. Build the locally vendored dependencies with automatic fetching turned on (or all of them if the system manager doesn't have dependency information).
3. Compile our project's code and dump the resulting image without the project manager code loaded.

This sounds easy enough, but let's explore the APIs and mechanics of doing this.

## Build tooling

ASDF is the de-facto build system of CL, and acts sort like Java's old-school build tool, Ant. It's responsible for building the individual components of a system, but can't download any dependencies it can't find. To do that, we need a *system manager*[^system-manager]. There are lots to choose from, including CLPM, qlot, quicklisp, and ocicl. Which one is used affects some details of this process; to make things easier, let's assume that the system manager has the proper dependency trees built in.

## Step 1: Finding Dependencies

To determine what systems are available, ASDF uses a *source-registry*. This lays out the search paths for the system, and is a list of directories and directory trees. To find all of our systems, we need to make sure that it includes the following locations:

+ The location of our target system
+ The location of any vendored dependencies
+ The location where our system manager downloads systems to.

Once we setup the source registry, we can use `asdf:find-system` to load our target system. With the system object returned by that function, we can recursively call `asdf:system-depends-on` and `asdf:find-system` on each dependency to determine what systems are present and which ones need to be downloaded. How deeply we go and for which systems depends on the system manager chosen.

There are a few things we need to take care of before we start building our list of missing systems. First, if we loaded the current code via ASDF, we need to remove it and its dependencies from the ASDF cache:

``` lisp
;; code goes here
```

Secondly, ASDF systems can require other systems at definition time. We need to install them as we traverse the dependency tree, which leads to a satisfying use of CL's signal system:

``` lisp
;; code goes here
```

Once we have found all the missing systems, we can download the first batch of dependencies from our system manager.

## Step 2: Dealing with Vendored Dependencies

To capture the systems loaded during the compilation of the vendored dependencies, we need to hook into ASDF's mechanism for finding systems:

``` lisp
;; code goes here
```

Once done, we can load the systems using `asdf:load-system`. After that process finishes, we should finally be ready to build our target system.

We are now done with the current lisp process, and it's time for building our actual executable.

## Step 3: Building the system

We can now kickoff the build process. First, we need to setup the same source registry as we did in step 1. If we dumped the configuration to a file, we can just load it. After calling `asdf:make`, we will have our final executable.

``` lisp
(load "init-build-env.lisp")
(asdf:make "our-system-name")
```

When loading this file, it's important to run it from a fresh environment with no init file loaded. With SBCL, that looks like this:

``` bash
sbcl --no-userinit --script $BUILD_SCRIPT
```

## What's Next

That's quite a lot for what is normally a simpler process, but it's all with the goal of making a lisp application as easy to install as what we would do with more modern build systems. The goal is to be able to perform a similar `./configure`, `make`, `make install` process that we can use with other languages.

The next step is to generate a ninja build file that can run our build step and keep everything up to date as our files change.

[^final-image]: Using the described process, the final image will still include ASDF. Removing would involve a similar process, but with gathering all the required files instead of just the dependencies. Due to ASDF being able to process non-CL targets, you'd probably need to avoid using it in the target system and use a build system that loads files by calling a lisp subprocesses.

[^system-manager]: Most other languages would call this a package manager, but since `package` means something different in CL, we avoid that term.