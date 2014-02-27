Problem
-------

Some project B task depends on running project A.
We can not declare dependency directly, because re-start
task requires arguments.


Solution
--------

We can define task without arguments, and depend on it:

      lazy val reStart0 = TaskKey[AppProcess]("re-start-0")
      lazy val emptyArgs = SettingKey[revolver.Actions.ExtraCmdLineOptions]("empty-args")

      lazy val projectA = Project(
        id = "hello-a",
        base = file("./a"),
        settings = Project.defaultSettings ++ Revolver.settings
      ).settings(
        emptyArgs := revolver.Actions.ExtraCmdLineOptions(Nil, Nil),
        reStart0 <<= {
        (streams, Revolver.reLogTag, thisProjectRef, Revolver.reForkOptions, mainClass in Revolver.reStart, fullClasspath in Runtime, Revolver.reStartArgs, emptyArgs)
           .map(revolver.Actions.restartApp)
           .dependsOn(products in Compile)
        }
      )

      lazy val projectB = Project(
        id = "hello-b",
        base = file("./b"),
        settings = Project.defaultSettings ++ Revolver.settings ++ Defaults.itSettings)
      .configs(IntegrationTest)
      .settings(
        test in (IntegrationTest) <<= (test in IntegrationTest).dependsOn(reStart0 in projectA)
      )