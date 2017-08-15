# First steps with SBT
## Référence site

[SBT reference site](http://www.scala-sbt.org/1.0/docs/Setup.html)

## Install SBT on my Mac

    brew install sbt

To check if SBT is well installed.

   sbt sbtVersion

**Warning** SBT has no usual `--version`.

You have two new directories :

    ~/.sbt
    ~/.ivy2

## Create a first project Hello, World

### SBT new comand

Go in the directory to host the scala new project and type the following command :

    sbt new sbt/scala-seed.g8

**Warning** You can't specify the new project name on the SBT new command line, instead you need to indicate `sbt new sbt/scala-seed.g8`.

To indicate the project name, wait the following prompt :

    Minimum Scala build. 

    name [My Something Project]: hello

When the project is created :

    Template applied in ./hello

In the current directory you have two subdirectories created :

    hello
    target

### Running hello app

Go in the directory `hello` and launch SBT :

    cd hello
    sbt
    ...
    > run
    ...
    [info] Compiling 1 Scala source to ~/Development/experiments/Scala/hello/target/scala-2.12/classes...
    [info] 'compiler-interface' not yet compiled for Scala 2.12.1. Compiling...
    [info]   Compilation completed in 9.777 s
    [info] Running example.Hello 
    hello
    [success] Total time: 40 s, completed 

### Leave SBT

To leave SBT shell `exit` or use `Ctrl+D` (Unix) or `Ctrl+Z` (Windows).

    > exit

## Project directory structure

### Base direcctory

The base directory is the directory containing the project.

### Source code

The code source use a Maven source directory structure relative of the base directory.

    ./src/main/scala
      +- /example/Hello.scala
    ./src/main/resources
    ./src/test/scala
      +- /example/HelloSpec.scala
    ./src/test/resources

### SBT build definition files

The build definition is described in `build.sbt` in the project’s base directory.

    build.sbt

### Build support files

A `project` directory can contain `.scala` files that defines helper objects and one-off plugins.

    project/Dependencies.scala

### Build products

Generated files (compiled classes, packaged jars, managed files, caches, and documentation) will be written to the target directory by default.

    ./target

### Configuring control version with git

Ingore all target directories with the following `.gitignore` content :

    target/
