Buildpack: Java
===============

This is a [Buildpack](http://doc.scalingo.com/buildpacks) for Java apps.
It uses Maven 3.3.9 to build your application and OpenJDK 8 to run it. However, the JDK version can be configured as described below.

## How it works

The buildpack will detect your app as Java if it has a `pom.xml` file in its root directory.  It will use Maven to execute the build defined by your `pom.xml` and download your dependencies. The `.m2` folder (local maven repository) will be cached between builds for faster dependency resolution. However neither the mvn executable or the .m2 folder will be available in your slug at runtime.

```
  $ ls
  Procfile  pom.xml  src

  $ scalingo create java-app

  $ git push scalingo master
  ...
  -----> Java app detected
  -----> Installing OpenJDK 1.8... done
  -----> Installing Maven 3.2.5... done
  -----> Installing settings.xml... done
  -----> executing /app/tmp/repo.git/.cache/.maven/bin/mvn -B -Duser.home=/tmp/build_19z6l4hp57wqm -Dmaven.repo.local=/app/tmp/repo.git/.cache/.m2/repository -s /app/tmp/repo.git/.cache/.m2/settings.xml -DskipTests=true clean install
         [INFO] Scanning for projects...
         [INFO]
         [INFO] ------------------------------------------------------------------------
         [INFO] Building readmeTest 1.0-SNAPSHOT
         [INFO] ------------------------------------------------------------------------
```

## Configuration

### Choose a JDK

Create a `system.properties` file in the root of your project directory and set `java.runtime.version=1.8`.

Example:

    $ ls
    Procfile pom.xml src

    $ echo "java.runtime.version=1.8" > system.properties

    $ git add system.properties && git commit -m "Java 8"

    $ git push scalingo master
    ...
    -----> Java app detected
    -----> Installing OpenJDK 1.8... done
    -----> Installing Maven 3.3.9... done
    ...

### Choose a Maven Version

The `system.properties` file also allows for `maven.version` entry
(regardless of whether you specify a `java.runtime.version` entry). For example:

```
java.runtime.version=1.8
maven.version=3.1.1
```

Supported versions of Maven include 3.0.5, 3.1.1, 3.2.5 and 3.3.9. You can request new
versions of Maven by submitting a pull request against `vendor/maven/sources.txt`.

### Customize Maven

There are three config variables that can be used to customize the Maven execution:

+ `MAVEN_CUSTOM_GOALS`: set to `clean dependency:list install` by default
+ `MAVEN_CUSTOM_OPTS`: set to `-DskipTests` by default
+ `MAVEN_JAVA_OPTS`: set to `-Xmx1024m` by default

These variables can be set like this:

```sh-session
$ scalingo env-set MAVEN_CUSTOM_GOALS="clean package" 
$ scalingo env-unset MAVEN_CUSTOM_OPTS="--update-snapshots -DskipTests=true"
$ scalingo env-set MAVEN_JAVA_OPTS="-Xss2g"
```

## Install Java and nothing else (no maven)

In the case of a combination of buildpack, you may want to install a custom
version of the JDK/JRE in addition to the other technologies you are using.

Use the branch `javaonly` for this purpose, it will always accept to threat
your app as a Java application and will only install the JDK and stop the build
there.

In the case of the use of the [multi buildpack](http://doc.scalingo.com/buildpacks/multi/),
your `.buildpacks` file should look like:

```
https://github.com/Scalingo/java-buildpack#javaonly
https://github.com/Scalingo/python-buildpack
```

Change python with the technology you are using.

## Development

To make changes to this buildpack, fork it on Github. Push up changes to your fork, then create a new Scalingo app to test it, or configure an existing app to use your buildpack:

```
# Create a new Scalingo app that uses your buildpack
scalingo create new-app

# Configure an existing Scalingo app to use your buildpack
scalingo env-set GITHUB_URL=<your-github-url>

# You can also use a git branch!
scalingo env-set GITHUB_URL=<your-github-url>#your-branch
```

For example if you want to have maven available to use at runtime in your application, you can copy it from the cache directory to the build directory by adding the following lines to the compile script:

    for DIR in ".m2" ".maven" ; do
      cp -r $CACHE_DIR/$DIR $BUILD_DIR/$DIR
    done

This will copy the local maven repo and maven binaries into your slug.

Commit and push the changes to your buildpack to your Github fork, then push your sample app to Scalingo to test. Once the push succeeds you should be able to run:

    $ scalingo run bash

and then:

    $ ls -al

and you'll see the `.m2` and `.maven` directories are now present in your slug.

License
-------

Licensed under the MIT License. See LICENSE file.
