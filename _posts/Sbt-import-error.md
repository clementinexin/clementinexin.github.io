
```scala
[error] (*:update) sbt.ResolveException: unresolved dependency: com.typesafe.play#sbt-plugin;2.5.5: not found
[error] unresolved dependency: com.typesafe.sbt#sbt-digest;1.1.0: not found
[error] unresolved dependency: com.typesafe.sbt#sbt-gzip;1.0.0: not found
```

- [](http://stackoverflow.com/questions/35021382/sbt-resolveexception-unresolved-dependency-com-typesafe-akkaakka-sbt-plugin2)

- plugins.sbt
```plain
// Comment to get more information during initialization
logLevel := Level.Warn

// The Typesafe repository
resolvers += "Typesafe repository" at "http://repo.typesafe.com/typesafe/releases/"

// Use the Play sbt plugin for Play projects
addSbtPlugin("com.typesafe.play" % "sbt-plugin" % "2.5.5")

addSbtPlugin("com.typesafe.sbt" % "sbt-digest" % "1.1.0")

addSbtPlugin("com.typesafe.sbt" % "sbt-gzip" % "1.0.0")
```

-  project file system
```plain
.
├── README.md
├── admin
│   ├── app
│   ├── conf
│   ├── public
│   └── test
├── common
│   └── src
├── docs
│   ├── awesome-play.asta
│   └── test-object.sql
├── images
│   ├── awesome-play.png
│   ├── awesome-play2.png
│   └── awesome-play3.png
├── order-center
│   └── src
└── project
    ├── Build.scala
    ├── Dependencies.scala
    ├── build.properties
    ├── commons.scala
    ├── plugins.sbt
    ├── project
    └── target
```

```plain
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=768M; support was removed in 8.0
[info] Loading project definition from /Users/daijiajia/Project/awesome-play/project
[error] /Users/daijiajia/Project/awesome-play/project/Dependencies.scala:1: object sbt is not a member of package play
[error] import play.sbt.PlayImport._
[error]             ^
[error] /Users/daijiajia/Project/awesome-play/project/Build.scala:28: object sbt is not a member of package play
[error]     enablePlugins(play.sbt.PlayJava).
[error]                        ^
[error] /Users/daijiajia/Project/awesome-play/project/Dependencies.scala:32: not found: value javaCore
[error]     javaCore,
[error]     ^
[error] /Users/daijiajia/Project/awesome-play/project/Dependencies.scala:33: not found: value javaWs
[error]     javaWs,
[error]     ^
[error] /Users/daijiajia/Project/awesome-play/project/Dependencies.scala:34: not found: value cache
[error]     cache,
[error]     ^
[error] /Users/daijiajia/Project/awesome-play/project/Dependencies.scala:35: not found: value javaJdbc
[error]     javaJdbc,
[error]     ^
[error] 6 errors found
[error] (compile:compileIncremental) Compilation failed
```

- [sbt-laucher.jar重新打包](http://dblab.xmu.edu.cn/blog/maven-network-problem/)
