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

Setting up Leiningen to [read GPG credentials](https://github.com/technomancy/leiningen/blob/master/doc/DEPLOY.md#authentication). Initially installed `gpg` and `gpg-agent` with homebrew. Then read the [Lein GPG guide](https://github.com/technomancy/leiningen/blob/stable/doc/GPG.md) to get help on generating suitable keypairs.

GPG keys generated. Encrypted the `~/.lein/credentials.clj` settings file, which holds the username and password credentials for my.datomic.com.

Added a dependency on Datomic in the [project.clj](./project.clj) file:
```clojure
  :dependencies [
                 [org.clojure/clojure "1.6.0"]
                 [com.datomic/datomic-pro "0.9.5067"]
                 ]
```

Had some problems with the GPG authentication when starting `lein repl`. Apparently one needs to run the `gpg-agent` and specify the `use-agent` configuration option in `~/.gnupg/gpg.conf`.

Weird. This [strange incantation](https://github.com/technomancy/leiningen/issues/1349#issuecomment-74310781) seemed to allow me to decrypt the credentials file:

```bash
killall ssh-agent gpg-agent
unset GPG_AGENT_INFO SSH_AGENT_PID SSH_AUTH_SOCK
eval $(gpg-agent --daemon --enable-ssh-support)
```

Next problem, dowloading the the datomic dependencies via lein threw this error:

```bash
Could not transfer artifact com.datomic:datomic-pro:pom:0.9.5067 from/to my.datomic.com (https://my.datomic.com/repo): Not authorized , ReasonPhrase:Unauthorized.
```

Logged in to my.datomic.com to check my password, which works for logging into the website, but also found a note saying that passcode for downloading the datomic dependencies automatically is different. Updated `~/.lein/credentials.clj` and re-encrypted, like so:

```bash
gpg --default-recipient-self -e \
~/.lein/credentials.clj > ~/.lein/credentials.clj.gpg
```

And running `lein repl` this time seemed to work:

```bash
Retrieving com/datomic/datomic-pro/0.9.5067/datomic-pro-0.9.5067.pom from my.datomic.com
```

Back to the tutorial...

## Connecting to the Memory database

This will mostly be a straight translation of the tutorial code, with comments where things aren't obvious.

```clojure
;; Import the Datomic Peer library (usable as 'Peer' from here on)
(import datomic.Peer)

;; Create the in memory database, calling the static class method Peer#createDatabase
(def uri "datomic:mem://hello")
(Peer/createDatabase uri)

;; Create a "connection" to the database
(def conn (Peer/connect uri))


;; Create a datom to describe the first bits of data we're entering into the DB:
(def datom ["db/add" (Peer/tempid "db.part/user") "db/doc" "hello world"])

;; Pass the datom to the transactor via the connection (note the 'vector of vectors' for the datom):
(def resp (.transact conn [datom]))

;; currently stuck here:
datomic-tutorial.core=> (Peer/query "[:find ?entity :where [?entity :db/doc \"hello world\"]]" db)

ClassCastException datomic.db.Db cannot be cast to [Ljava.lang.Object;
```
