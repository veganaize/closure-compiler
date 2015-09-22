##Initial Setup
To setup a project you must have Gradle installed https://gradle.org

Init the project:
```
/some/path$> mkdir my-project && cd my-project
/some/path/my-project$> gradle init
```
Gradle inits a _build.gradle_ where the main configuration is happening, further it creates an executable _gradlew_ / _gradlew.bat_ that can be checked into your VCS. This tiny script so called Gradle Wrapper downloads Gradle locally into your project and ensures that all your team members work with the same Gradle version.

##Gradle as closure compiler command line wrapper
###Setup

*build.gradle*

```
repositories {
  mavenCentral() //or jcenter()
}
configurations {
  closureCompiler
}
dependencies {
  closureCompiler 'com.google.javascript:closure-compiler:v20150609'
}

task compileJS(type: JavaExec){
  classpath configurations.closureCompiler
  main = 'com.google.javascript.jscomp.CommandLineRunner'

  def closureArgs = []
  //append all your command line options here
  closureArgs << "--compilation_level=SIMPLE_OPTIMIZATIONS"
  closureArgs << "--js_output_file=app.js"
  closureArgs << "input1.js"
  closureArgs << "input2.js"
  closureArgs << "src/**.js"

  args closureArgs
}
```
###Run
```./gradlew compileJS```

##Gradle Plugin - [gradle-js-plugin](http://eriwen.github.io/gradle-js-plugin/)
Combines JavaScript sources and minifies it with the [closure compiler](https://github.com/eriwen/gradle-js-plugin#minifyjs-uses-the-google-closure-compiler)
###Setup
*build.gradle*
```
plugins {
  id "com.eriwen.gradle.js" version "1.12.1"
}
javascript.source {
    dev {
        js {
            srcDir "src"
            include "*.js"
            exclude "*.min.js"
        }
    }
}
combineJs {

    source = javascript.source.dev.js.files
    dest = file("${buildDir}/all.js")
}

minifyJs {
    source = combineJs
    dest = file("${buildDir}/all-min.js")
    sourceMap = file("${buildDir}/all.sourcemap.json")
    closure {
        warningLevel = 'QUIET'
    }
}
```
###Run
```./gradlew minifyJs```

##Gradle Plugin - [silksmith](http://silksmith.io/)
The silksmith plugin contains similar to plovr a development server that serves all your closure javascript, precompiled libs like jquery and generates the _deps.js_ for you on the fly. Further it allows you to specify dependencies to other libraries that may contain closure js sources or be already precompiled and have externs, silksmith will configure this for you. It also comes with a built-in test runner, that setups the mocha suite and executes the provided test.

###Setup
Currently requires Java 8

*build.gradle*
```

plugins {
  id "io.silksmith.plugin" version "0.5.0"
}
repositories {
    jcenter() //currently needed for jruby
    maven { url="http://dl.bintray.com/silksmith-io/silk"}
}
dependencies {

    web "io.silksmith.libs:closure-base:1.0.2+smith.+"

    //lets use jquery
    web "io.silksmith.libs:jquery:1.11.2+smith.+"

    //some test dependencies
    testWeb "io.silksmith.libs:chai:2.3.0+smith.0"

}
closureCompileJS{
    entryPoint="your.entryPoint"
}
server {
    dir file("www") //let's say your host index.html is in www
}
```
js/html part:
- put all your closure js code in ```src/main/js```
- put all your test js code in ```src/main/js```
- in your ```www/index.html``` include a script tag ```<script src="your-app.js">``` (if not explicitly set the name is  **{project-folder-name}.js**)

###Run

####Developmenet
```./gradlew server```

Then open http://localhost:10101

####Test
```./gradlew testJS```
Executes the tests in FireFox (must be installed)

####Build
```./gradlew closureCompileJS```

Runs the closure compiler against your sources (output ```build/compiled/web/your-app.js```)

See this runnning mini todo app https://github.com/silksmith/silksmith-todo-sample

(Note: I'm the author of silksmith)
