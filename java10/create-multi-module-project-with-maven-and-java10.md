# Create multi modules project with maven et Java 10

* Project `todoListInJava10` created
    * JDK 10 selected
    * groupId set to `experiments`
    * artifactId set to `todoListInJava10`
    * Add `pom.xml`
        * Packaging set to `pom`
        * Compiler source set to 10
        * Compiler target set to 10
        * Source encoding set to UTF-8
        * Reporting encoding set to UTF-8
        * Add modules `messages` and `domain`
        * Add `messages` module managed dependency
        * Add jetbrains annotations managed dependency
        * Add `junit 5` managed dependency
        * Add `assertj` managed dependency
        * Add `maven compiler 3.7.0` plugin with `asm 6.1.1` dependency to fix maven compile error
        * Add `surfire 2.1.19` plugin with `junit-platform-surefire-provider` to run `junit 5` tests
    * Add module `messages`
        * Add sources root `src/main/java` and `module-info.java`
        * Add resources root `src/main/resources`
        * Add test sources root `src/test/java`
        * Add test resources root `src/test/resources`
        * Add `pom.xml`
            * Add parent of module `todoListInJava10`
            * Add jetbrains annotations dependency
    * Add module `domain`
        * Add sources root `src/main/java` and `module-info.java`
        * Add resources root `src/main/resources`
        * Add test sources root `src/test/java`
        * Add test resources root `src/test/resources`
        * Add `pom.xml`
            * Add parent of module `todoListInJava10`
            * Add jetbrains annotations dependency
            * Add `messages` module dependency
            * Add `junit 5` dependency
    * Add `.gitignore`
        * Ignore `target`
        * Ignore `.idea`
        * Ignore `*.iml`
    * Add `default.nix`
        * Run maven with JDK 10 in `nix-shell`
