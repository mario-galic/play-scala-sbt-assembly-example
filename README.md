# play-scala-sbt-assembly-example

This is a working example of how to deploy Play app using `sbt-assembly` instead of using `sbt-native-packager`.
It is based on [playframework/play-scala-starter-example](https://github.com/playframework/play-scala-starter-example)
and [Play docs](https://www.playframework.com/documentation/2.7.x/Deploying#Using-the-SBT-assembly-plugin)

Also, it attempts to answer Stack Overflow question [Merge Strategy in sbt assembly and missing application loader](https://stackoverflow.com/questions/49414413/merge-strategy-in-sbt-assembly-and-missing-application-loader)

Deploying:
1. Add `sbt-assembly` to `project/plugins.sbt`:
    ```
    addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.5")
    ```
1. Exclude docs from package:
    ```
    sources in (Compile, doc) := Seq.empty
    publishArtifact in (Compile, packageDoc) := false
    ```
1. Define `assemblyMergeStrategy`
    ```
    assemblyMergeStrategy in assembly := {
      case manifest if manifest.contains("MANIFEST.MF") =>
        // We don't need manifest files since sbt-assembly will create
        // one with the given settings
        MergeStrategy.discard
      case referenceOverrides if referenceOverrides.contains("reference-overrides.conf") =>
        // Keep the content for all reference-overrides.conf files
        MergeStrategy.concat
      case x =>
        // For all the other files, use the default sbt-assembly merge strategy
        val oldStrategy = (assemblyMergeStrategy in assembly).value
        oldStrategy(x)
    }
    ```
1. Package with `sbt assembly` which should create a package in:
    ```
    /Users/mario/IdeaProjects/play-scala-sbt-assembly-example/target/scala-2.12/play-scala-sbt-assembly-example-assembly-1.0-SNAPSHOT.jar
    ```
1. Run with 
    ```
    java -Dplay.http.secret.key=ad31779d4ee49d5ad5162bf1429c32e2e9933f3 -jar /Users/mario/IdeaProjects/play-scala-sbt-assembly-example/target/scala-2.12/play-scala-sbt-assembly-example-assembly-1.0-SNAPSHOT.jar
    ```
1. Test by hitting [localhost:9000/](http://localhost:9000)

