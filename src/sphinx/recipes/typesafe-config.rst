Example: Typesafe Config Library
--------------------------------

Now that we have ability to configure the JVM, let's add in a more robust method of customizing the application.  We'll be using the `Typesafe Config <https://github.com/typesafehub/config>`_ library for this purpose.

First, let's add it as a dependency in ``build.sbt`` ::

   libraryDependencies += "com.typesafe" % "config" % "1.2.0"

Next, let's create the configuration file itself.  Add the following to ``src/universal/conf/app.config`` ::

    example {
      greeting = "Hello, World!"
    }

Now, we need a means of telling the typesafe config library where to find our configuration.  The library supports
a JVM property "``config.file``" which it will use to look for configuration.   Let's expose this file
in the startup BASH script.  To do so, add the following to ``build.sbt`` ::

    bashScriptExtraDefines += """addJava "-Dconfig.file=${app_home}/../conf/app.config""""

This line modifies the generated BASH script to add the JVM options the location of the application configuration on disk.  Now, let's modify the application (``src/main/scala/TestApp.scala``) to read this configuration

.. code-block:: scala

    import com.typesafe.config.ConfigFactory

    object TestApp extends App {
      val config = ConfigFactory.load()
      println(config.getString("example.greeting"))
    }

Now, let's try it out on the command line ::

    $ sbt stage
    $ ./target/universal/stage/bin/example-cli
    Hello, World!


Finally, let's see what this configuration looks like in a linux distribution.  Let's run the debian packaging again ::

    $ sbt debian:packageBin

The resulting structure is the following ::

    /usr/
      share/example-cli/
        conf/
          app.config
          application.ini
        bin/
          example-cli
        lib/
          example-cli.example-cli-1.0.jar
          org.scala-lang.scala-library-2.10.3.jar
      bin/
        example-cli -> ../share/example-cli/bin/example-cli
    /etc/
       example-cli -> /usr/share/example-cli/conf

Here, we can see that the entire ``conf`` directory for the application is exposed on ``/etc`` as is standard for
other linux applications.  By convention, all files in the universal ``conf`` directory are marked as configuration
files when packaged, allowing users to modify them.
