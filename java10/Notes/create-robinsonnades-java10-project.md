# Create robinsonnades java 10 Project

## Create `robinsonnades-java10` maven project

* `groupId: experiments``
* `artifactId: robinsonnades-java10``

### Add `.gitignore`

* Ignore `target`

```zsh
echo 'target' >> .gitignore
```

* Ignore `.idea`

```sh
echo '.idea' >> .gitignore
```

* Ignore `*.iml`

```sh
echo '*.iml' >> .gitignore
```

### Init git repository

```sh
git Init
```

### Set maven project properties

* Set source and target java version

```xml
<maven.compiler.source>10</maven.compiler.source>
<maven.compiler.target>10</maven.compiler.target>
```

* Set sources and reporting encoding

```sh
<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
<project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
```

### Add `default.nix`

```nix
with import <nixpkgs> {};
let
mvn = maven.override { jdk="/Library/Java/JavaVirtualMachines/jdk-10.jdk/Contents/Home/"; };
in
stdenv.mkDerivation {
   name = "mvn-java10";
   JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk-10.jdk/Contents/Home/";
   buildInputs = [mvn];
}
```

## Add module `reads`

> artifactId: reads
