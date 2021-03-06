# spark-class shell script

`spark-class` shell script is the Spark application command-line launcher that is responsible for setting up JVM environment and executing a Spark application.

NOTE: Ultimately, any shell script in Spark, e.g. link:spark-submit.adoc[spark-submit], calls `spark-class` script.

You can find `spark-class` script in `bin` directory of the Spark distribution.

When started, `spark-class` first loads `$SPARK_HOME/bin/load-spark-env.sh`, collects the Spark assembly jars, and executes <<main, org.apache.spark.launcher.Main>>.

Depending on the Spark distribution (or rather lack thereof), i.e. whether `RELEASE` file exists or not, it sets `SPARK_JARS_DIR` environment variable to `[SPARK_HOME]/jars` or `[SPARK_HOME]/assembly/target/scala-[SPARK_SCALA_VERSION]/jars`, respectively (with the latter being a local build).

If `SPARK_JARS_DIR` does not exist, `spark-class` prints the following error message and exits with the code `1`.

```
Failed to find Spark jars directory ([SPARK_JARS_DIR]).
You need to build Spark with the target "package" before running this program.
```

`spark-class` sets `LAUNCH_CLASSPATH` environment variable to include all the jars under `SPARK_JARS_DIR`.

If `SPARK_PREPEND_CLASSES` is enabled, `[SPARK_HOME]/launcher/target/scala-[SPARK_SCALA_VERSION]/classes` directory is added to `LAUNCH_CLASSPATH` as the first entry.

NOTE: Use `SPARK_PREPEND_CLASSES` to have the Spark launcher classes (from `[SPARK_HOME]/launcher/target/scala-[SPARK_SCALA_VERSION]/classes`) to appear before the other Spark assembly jars. It is useful for development so your changes don't require rebuilding Spark again.

`SPARK_TESTING` and `SPARK_SQL_TESTING` environment variables enable *test special mode*.

CAUTION: FIXME What's so special about the env vars?

`spark-class` uses <<main, org.apache.spark.launcher.Main>> command-line application to compute the Spark command to launch. The `Main` class programmatically computes the command that `spark-class` executes afterwards.

TIP: Use `JAVA_HOME` to point at the JVM to use.

=== [[main]] Launching org.apache.spark.launcher.Main Standalone Application

`org.apache.spark.launcher.Main` is a Scala standalone application used in `spark-class` to prepare the Spark command to execute.

`Main` expects that the first parameter is the class name that is the "operation mode":

1. `org.apache.spark.deploy.SparkSubmit` -- `Main` uses link:spark-submit-SparkSubmitCommandBuilder.adoc[SparkSubmitCommandBuilder] to parse command-line arguments. This is the mode link:spark-submit.adoc[spark-submit] uses.
2. _anything_ -- `Main` uses `SparkClassCommandBuilder` to parse command-line arguments.

```
$ ./bin/spark-class org.apache.spark.launcher.Main
Exception in thread "main" java.lang.IllegalArgumentException: Not enough arguments: missing class name.
	at org.apache.spark.launcher.CommandBuilderUtils.checkArgument(CommandBuilderUtils.java:241)
	at org.apache.spark.launcher.Main.main(Main.java:51)
```

`Main` uses `buildCommand` method on the builder to build a Spark command.

If `SPARK_PRINT_LAUNCH_COMMAND` environment variable is enabled, `Main` prints the final Spark command to standard error.

```
Spark Command: [cmd]
========================================
```

If on Windows it calls `prepareWindowsCommand` while on non-Windows OSes `prepareBashCommand` with tokens separated by `  \0`.

CAUTION: FIXME What's `prepareWindowsCommand`? `prepareBashCommand`?

`Main` uses the following environment variables:

* `SPARK_DAEMON_JAVA_OPTS` and `SPARK_MASTER_OPTS` to be added to the command line of the command.
* `SPARK_DAEMON_MEMORY` (default: `1g`) for `-Xms` and `-Xmx`.
