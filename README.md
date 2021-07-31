# Background
This repo is forked from UCSB, who updated the original Gradescope demo to use Maven and a JAR for jh61b. Changes are:

* Changed from Maven 3.6.0 to 3.8.1. 
* Changed from JUnit 4.12 to 5.7.2.
* Changed maven-compiler-plugin from 3.5.1 to 3.8.1.
* Changed maven target from Java 1.8 to 11. (Supposedly needed for JUnit 5.)
* Updated install scripts to automatically handle jh61b install.

Note that the Maven version change is to fix an issue with a reflective call made during run_autograder.

-RA

# To install jar for jh61b, use:

Use:

```
mvn install:install-file -Dfile=../jh61b/target/jh61b-1.0.jar
mvn package
mvn -q exec:java
```

The install command is described here, and is the way you get a third party
JAR into Maven.

* <https://maven.apache.org/guides/mini/guide-3rd-party-jars-local.html>

The rest is described below.

You also need to adjust this part of the pom.xml.  The main class needs to be a `RunTests` class (sample source code below.)

```
      <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.4.0</versio>
        <executions>
          <execution>
            <goals>
              <goal>java</goal>
            </goals>
          </execution>
        </executions>
        <configuration>
          <mainClass>com.gradescope.intlist.tests.RunTests</mainClass>
          <arguments>
          </arguments>
        </configuration>
      </plugin>

```

Sample Source for `RunTests` (which can be in any package you like):

The part you have to modify is the list of test classes in
the `@Suite.SuiteClasses` annotation.  Only those classes get picked up
by the grading stuff.

Then you also need to annotate your classes as shown in the sample classes
in this repo.


```
package com.gradescope.intlist.tests;

import org.junit.runner.RunWith;
import org.junit.runners.Suite;
import org.junit.runner.JUnitCore;
import org.junit.runner.Result;
// import junit.tests.framework.TestListenerTest
import com.gradescope.intlist.tests.IntListTest;
import com.gradescope.jh61b.grader.GradedTestListenerJSON;

@RunWith(Suite.class)
@Suite.SuiteClasses({
        IntListTest.class,
        IntListPredicates.class,
})
public class RunTests {
    public static void main(String[] args) {
        JUnitCore runner = new JUnitCore();
        runner.addListener(new GradedTestListenerJSON());
        // runner.addListener(new TestListenerTest());
        Result r = runner.run(RunTests.class);
    }
}
```


# Gradescope Java+Maven Autograder Example

This example shows how to set up an autograder on Gradescope for a
Java project using Maven. For the full Java example, see
[here](../java). This document only describes the differences needed
for Maven.

## Maven

Maven is a build tool. It provides dependency resolution (only needed
for JUnit and exec plugin in this project) and a way to compile and
execute code (using exec plugin). It may be useful if you don't want
to have to package your dependencies (though that might be a better
idea for repeatability reasons), want better handling of complex
build scenarios, or if you're already using Maven.


## Generating a Maven project

Maven projects are defined by a pom.xml file. This file and the
project structure were generated using the command

`mvn archetype:generate -DgroupId=com.gradescope.intlist -DartifactId=intlist -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false`

# Files

## [setup.sh](https://github.com/gradescope/autograder_samples/blob/master/java-mvn/11setup.sh)

Sets up JDK 8 and maven. There's a workaround for a ca-certificates
issue to make sure you can install things from Maven.

## [run_autograder](https://github.com/gradescope/autograder_samples/blob/master/java-mvn/run_autograder)

mvn compile is a built in maven goal to compile the project. -q makes
it quiet (i.e. not spam a bunch of debugging output).

mvn exec:java is a goal from the exec plugin for maven. It lets you
specify a main class to execute (see pom.xml). This lets you avoid
having to figure out the proper java command to run and classpath
arguments and the like.

## [pom.xml](https://github.com/gradescope/autograder_samples/blob/master/java-mvn/pom.xml)

This is a maven project file. Briefly, it describes the project, its
dependencies/plugins, Java compiler version, and main class.


# Gradescope Java Autograder Example

[View project source on Github](https://github.com/gradescope/autograder_samples/tree/master/java)

This example shows how to set up an autograder on Gradescope for a
Java project. It uses JUnit, JDK8, and Josh Hug's jh61b library for
producing output in JSON format.

## Alternatives

You may want to check out
[JGrade](https://github.com/tkutche1/jgrade), a library created by JHU
instructors, which is more actively maintained.

## Project Description

This project builds a simple Linked List that stores ints as data
values.

## Explanation of Dependencies

- JUnit: Popular Java unit testing framework
- jh61b: Among other things, this provides annotations for JUnit tests that allows setting point values and names, and a test listener that produces output in JSON format
- JDK8: jh61b uses String.join() which is added in JDK8. If you need versions <8, you just need to replace this part of the code

# Files

## [setup.sh](https://github.com/gradescope/autograder_samples/blob/master/java/setup.sh)

Sets up OpenJDK 8.

Note: Installing JDK8 takes a few minutes, so building the image takes
a while. We may later provide base images to speed this up.

## [run_autograder](https://github.com/gradescope/autograder_samples/blob/master/java/run_autograder)

Copies the student's code to the autograder directory, compiles, and
executes it.

One thing to be aware of for Java projects is that you need to copy
the student's work to the right place depending on the
package. Depending on how you set up the project, students may submit
files in the root of their submission or within nested directories for
the package they used. Either is fine, you just need to sure that when
you're copying the files around you put them in the right place. Your
autograder code should know what package to import from, so make sure
that you know ahead of time what the student's package is (i.e. tell
them what to do or set up a template that has it filled in).

You could also do something like `$(find . -name "IntList.java")` in
your bash script and copy the result of that to the destination, but
it's probably better to just require them to submit in a certain
structure, which Gradescope will in the future make easier to verify.

This script calls compile.sh and run.sh to compile and run code.

### [compile.sh](https://github.com/gradescope/autograder_samples/blob/master/java/compile.sh)

This script finds all source files under the *src* directory and
compiles them. It adds the junit and hamcrest jars in the lib
directory to the classpath (hamcrest is a dependency of JUnit). It
produces output in the *classes* directory.

### [run.sh](https://github.com/gradescope/autograder_samples/blob/master/java/run.sh)

This script just runs the IntListTest class. It adds the compiled
classes and bundled libraries to the classpath.


## [AbstractIntList.java](https://github.com/gradescope/autograder_samples/blob/master/java/src/main/java/com/gradescope/intlist/AbstractIntList.java)

This is the abstract base class for the students' IntList
implementation. It's essentially a linked list for ints, with a few
operations you can do on it.  It has a constructor and .equals()
method implemented, so that you can rely on those things being
consistent in your tests.

## [RefIntList.java](https://github.com/gradescope/autograder_samples/blob/master/java/src/main/java/com/gradescope/intlist/RefIntList.java)

This is a "reference implementation" of the IntList class. One
possible approach to writing tests is to have your reference
implementation as part of your autograder, and compare the student's
return values to the reference implementation. Also, if you need to
rely on certain functions in the student's code working for some
tests, you should use your reference implementation to set up the test
data structures and then call the student's code. For Java, this may
require setting up a copy constructor in your students' class.

## [IntList.java](https://github.com/gradescope/autograder_samples/blob/master/java/src/main/java/com/gradescope/intlist/IntList.java)

This is the template file that you would give to students to fill
in. It has some parts filled in that should be kept by students; in
particular, the copy constructor is used in the tests to allow setting
up a test using the reference implementation and then copying the data
to an instance of the student's implementation so that you can test
individual functions in isolation instead of relying on students to
implement basic functionality correctly. Due to the way Java works
this can't be done in the abstract parent class.

## [IntList.java solution](https://github.com/gradescope/autograder_samples/blob/master/java/solution/IntList.java)

This is an example solution. It's just the reference implementation
with the name changed. You can submit this to Gradescope to see how it
works.

## [IntListTest.java](https://github.com/gradescope/autograder_samples/blob/master/java/src/main/java/com/gradescope/intlist/tests/IntListTest.java)

This is the actual Test class. It imports the necessary parts of
jh61b, JUnit, and the student's code. Then, it runs a set of basic
tests on the student's code.

## [IntListPredicates.java](https://github.com/gradescope/autograder_samples/blob/master/java/src/main/java/com/gradescope/intlist/tests/IntListPredicates.java)

This is another Test class, just to demonstrate multi-class test suites.

## [RunTests.java](https://github.com/gradescope/autograder_samples/blob/master/java/src/main/java/com/gradescope/intlist/tests/RunTests.java)

This class actually runs the tests. This demonstrates setting up a
Suite in JUnit.
