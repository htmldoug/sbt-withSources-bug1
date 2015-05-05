# sbt-withSources-bug1

Example project showing sbt's `withSources()` incorrectly exporting source code to downstream dependencies and artifacts.

## Steps
```
sbt publishLocal
open ~/.ivy2/local/default/sbt-withsources-bug1_2.10/0.1-SNAPSHOT/poms/sbt-withsources-bug1_2.10.pom
```

## Expected Behavior
Sources are downloaded but **not exported**.

Per http://www.scala-sbt.org/0.13/docs/Library-Management.html,

> To have sbt download the dependency’s sources without using an IDE plugin, add withSources() to the dependency definition. For API jars, add withJavadoc(). 

## Actual Behavior: Leaks source code

Can be reproduced by running `sbt publishLocal` and inspecting the resulting pom.xml.

In addition to downloading the sources, `withSources()` also exports the source jars in the pom! sbt-native-packager
in our play projects happily picks up the sources and **deploys our source code to our prod servers**. This sbt behavior
is undocumented and results in a security risk.

build.sbt:
```scala
libraryDependencies += "com.typesafe" % "config" % "1.2.1" withSources()
```

pom.xml:
```xml
<dependency>
    <groupId>com.typesafe</groupId>
    <artifactId>config</artifactId>
    <version>1.2.1</version>
    <classifier>sources</classifier>
</dependency>
<dependency>
    <groupId>com.typesafe</groupId>
    <artifactId>config</artifactId>
    <version>1.2.1</version>
</dependency>
```
