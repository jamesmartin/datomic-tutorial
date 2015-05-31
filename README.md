# Learning Datomic

Here lies my experience of working through the [Datomic tutorial](http://docs.datomic.com/tutorial.html).

My current system is a 2014 iMac, running OSX Yosemite (10.10). I plan to do the tutorial using Clojure, which is still very much a foreign language to me at present. I'm about half way through reading ["The Joy of Clojure"](http://www.joyofclojure.com), as I write this.

## Getting Datomic

Was a little confused by the distinction between the "free" and "Pro Starter" editions of Datomic, but the [Getting Started](http://docs.datomic.com/getting-started.html) guide recommended "Pro Starter", so I registered at [my.datomic.com](http://my.datomic.com) — which requires specifying an *organisation* — verified my email address and then downloaded the zip file.

A twelve month license key for my account was received in a subsequent email.

## Installation

Unzipped the datomic zip file (version datomic-pro-0.9.5173) and checked out
the ./bin and ./config/samples directories.

Tried to run `maven-install` to get the "peer library" installed locally, but discovered that I don't have Maven on my system.

Installed Maven via Homebrew;
```bash
brew install maven
```

Running `./bin maven-install` revealed [problems with the Java installation](http://stackoverflow.com/questions/29255495/maven-installation-osx-error-unsupported-major-minor-version-51-0) on this machine:

```bash
[datomic-pro-0.9.5173]$ ./bin/maven-install
Installing datomic-pro-0.9.5173 in local maven repository...
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/apache/maven/cli/MavenCli : Unsupported major.minor version 51.0
  at java.lang.ClassLoader.defineClass1(Native Method)
  ...
```

Checked the JDK version and found this:
```bash
[datomic-pro-0.9.5173]$ java -version
java version "1.6.0_65"
Java(TM) SE Runtime Environment (build 1.6.0_65-b14-466.1-11M4716)
Java HotSpot(TM) 64-Bit Server VM (build 20.65-b04-466.1, mixed mode)
```

Looks like this is JDK 6. Datomic requires JDK 7 or 8, so downloaded and installed JDK 8 from the Oracle website. After that was done, I tried running the `./bin/maven-install` command, which worked.


## Project Setup

The 'Getting Started' guide was a little confusing: Instructions are given for configuring both Maven and Leiningen, but it's unclear whether I need to setup both if I'm only going to be using Leiningen.

It seems that the `./bin/maven-install` part was just to install the "peer library" locally, however, in order to use the peer library as a project dependency, it's necessary to configure Maven (`pom.xml`) or Leiningen (`project.clj`) to use my.datomic.com login credentials.

I'm going to try doing both: The Maven install has already completed, so in theory I should be able to use the peer library locally. I'm setting up a separate Leiningen project (this repo) to use my Datomic credentials as well, so it could potentially be deployed to a standalone "production" system.

Read a bit of the [Leiningen Tutorial](https://github.com/technomancy/leiningen/blob/stable/doc/TUTORIAL.md#creating-a-project) and created a project with `lein new app datomic-tutorial`. Nice and easy.

Appreciated this line from the tutorial:

> If you come from the Java world, Leiningen could be thought of as "Maven meets Ant without the pain".

Hah!

