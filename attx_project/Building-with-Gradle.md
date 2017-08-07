# Building with (Py)Gradle

Brief report on interacting and working with Gradle within the project.

### Table of Contents
<!-- TOC START min:1 max:4 link:true update:true -->
- [Building with (Py)Gradle](#building-with-pygradle)
    - [Table of Contents](#table-of-contents)
  - [Gradle](#gradle)
    - [Installation](#installation)
    - [Setting up](#setting-up)
  - [Python + Gradle = ♥️](#python--gradle--)
    - [Setting up](#setting-up-1)
      - [Creating a PyPi repository](#creating-a-pypi-repository)
      - [Python project setup](#python-project-setup)
      - [Excluding depenencies](#excluding-depenencies)
    - [Building and Running Tasks](#building-and-running-tasks)
    - [Example PyGradle Projects](#example-pygradle-projects)
  - [Documentation](#documentation)

<!-- TOC END -->

## Gradle

### Installation

Installing on Ubuntu:
* `curl -s https://get.sdkman.io | bash` - install sdk
* `sdk install gradle 3.5` - install gradle 3.5 (or any version 3.0+)
* check the installed version `gradle -v`
* switching between versions `sdk use gradle 4.0`

Installing on MacOS (assuming [homebrew](http://brew.sh/) is installed):
* `brew install gradle`
* check the installed version `gradle -v`

### Setting up

There are two ways to work with Gradle:
* install it on your machine - see [Installation](#installation)
* provide a [wrapper](https://docs.gradle.org/current/userguide/gradle_wrapper.html) so that others can also work with the same version or we can distribute the project for others that do no have Gradle installed, or need to work with a specific version.

There are two ways to create a Gradle Wrapper (both require Gradle installed):
* use command `gradle wrapper --gradle-version 3.5` (last value is the version of Gradle set for the wrapper);
* in the `build.gradle` file create a wrapper task (see below) and use command `gradle wrapper`

```groovy
task wrapper(type: Wrapper) {
   gradleVersion = '3.5'
}
```

## Python + Gradle = ♥️

### Setting up

Setting up steps overview - all these steps are detailed in this section:

1. acquire Gradle and (optional) create a Gradle wrapper - see [Installation](#installation);
2. create PyPi repository - see [Creating a PyPi repository](#creating-a-pypi-repository);
3. set up Python project structure;
4. create `build.gradle` file with appropriate PyGradle plugin;
5. generate project `setup.py`.

One of the best way to work with Gradle in Python is using [PyGradle](https://github.com/linkedin/pygradle). It is also recommended using Gradle version 3.+ .

PyGradle comes with several plugins available (for specific details refer to PyGradle github repository).

| Plugin Name                                           | Used When                                     |
|-------------------------------------------------------|-----------------------------------------------|
| [com.linkedin.python](https://github.com/linkedin/pygradle/docs/python.md)                 | Base Python Plugin                            |
| [com.linkedin.python-sdist](https://github.com/linkedin/pygradle/docs/python-sdist.md)     | Developing Libraries                          |
| [com.linkedin.python-web-app](https://github.com/linkedin/pygradle/docs/python-web-app.md) | Developing Deployable Applications            |
| [com.linkedin.python-cli](https://github.com/linkedin/pygradle/docs/python-cli.md)         | Developing Command Line Applications          |
| [com.linkedin.python-flyer](https://github.com/linkedin/pygradle/docs/python-flyer.md)     | Developing Flyer (Flask + Ember) Applications |
| [com.linkedin.python-pex](https://github.com/linkedin/pygradle/docs/python-pex.md)         | Developing Pex Applications                   |

PyGradle depends on Ivy metadata to build a dependency graph, thus one cannot use pypi directly. For this reason it is recommended to build a local pypi repository using the [pivy-importer](https://github.com/linkedin/pygradle/tree/master/pivy-importer); for the latest pivy-importer jre see [pivy-importer jfrog bintray](https://bintray.com/linkedin/maven/pivy-importer).

The plugins can be added from Gradle plugins repository (https://plugins.gradle.org/):
```groovy
plugins {
  id "com.linkedin.python-cli" version "0.6.1"
}
```
or by jcenter
```groovy
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath group: 'com.linkedin.pygradle', name: 'pygradle-plugin', version: '0.6.1'
    }
}
apply plugin: 'com.linkedin.python-cli'
```

#### Creating a PyPi repository

There are three methods to provide a PyPi repository:
* local - that can be built following the instructions:
Requires the pivy repository locally check `build.gradle`
  * download from https://bintray.com/linkedin/maven/pivy-importer#files/com/linkedin/pygradle/pivy-importer/0.6.1
  * run as below

    ```shell
    java -jar ~/Software/pivy-importer-0.6.1-all.jar --repo /home/user/repository \
      virtualenv:15.0.1 \
      pip:7.1.2 \
      gunicorn:19.6.0 \
      lxml:3.6.4 rdflib:4.2.1 \
      rdflib-jsonld:0.4.0  \
      PyMySQL:0.7.9 \
      setuptools:19.1.1 \
      pytest:2.9.1 \
      pytest-cov:2.2.1 \
      pytest-xdist:1.14 \
      setuptools:32.1.0 \
      pytest-html:1.12.0  \
      --replace alabaster:0.7=alabaster:0.7.1, \
      pytz:0a=pytz:2016.4, \
      Babel:0.8=Babel:1.0, \
      sphinx_rtd_theme:0.1=sphinx_rtd_theme:0.1.1, \
      idna:2.0.0=idna:2.1, \
      chardet:2.2=chardet:2.3, \
      setuptools:0.6a2=setuptools:32.1.0

    ```
* hosted: one can build his own hosted PyPi repo for PyGradle using a [Docker image](https://github.com/ATTX-project/development-environment/tree/dev/pivy-repo) - build your own repository using https://github.com/blankdots/ivy-pypi-repo

* pypi server as detailed here [PyGradle pipConfig](https://github.com/linkedin/pygradle/blob/master/docs/plugins/python.md#pipconfig-examples)
```groovy
python{
     pipConfig = ['global':['index-url': 'https://<login>:<password>@your.repo.com/custom/url', 'timeout': '60']]
}
```

#### Python project setup

For a full example and detailed description visit: https://github.com/linkedin/pygradle/tree/master/examples/example-project .

Standard python project tree -  *Note the lack of a `setup.py`, more on this in a moment*:

```config
.
├── README.md
├── setup.cfg
├── src
│   └── project
│       └── __init__.py
│       └── example.py
└── test
    └── test_example.py
```


Create a `build.gradle` file. An example of a such a file is bellow with some specific options.

```groovy
plugins {
  id "com.linkedin.python-cli" version "0.6.1"
}

python {
  // force to use a specific version for a plugin
  forceVersion('pypi', 'pip', '7.1.2')
  // set the test directory to be used by py.test
  // by default it is test directory.
  testDir = file('tests')

  // Use a specific python version if it is available in the PATH it
  // will use it
  details.prependExecutableDirectory(new File('/usr/bin/python3.4'))
  // or details.appendExecutableDirectory(new File('/usr/bin/python3.4'))
  details.pythonVersion = '3.4'
}

// define an explicit installation sequence for the dependencies.
project.tasks.findByName('installPythonRequirements').sorted = false

dependencies {
    python 'pypi:requests:2.9.1'
    test 'pypi:mock:1.0.1'
}

repositories {
  // provided by LinkedIn as an example
   pyGradlePyPi()

   // using a local repository
   ivy{
     name 'pypi-local'
     url "/home/user/pivy"
     layout 'pattern' , {
       artifact '[organisation]/[module]/[revision]/[artifact]-[revision](-[classifier]).[ext]'
       ivy '[organisation]/[module]/[revision]/[module]-[revision].ivy'
     }
   }
}
```

Create the `setup.py` using `gradle generateSetupPy`

The `setup.py` in this example was created using the above task, by default the "setup" section at the bottom is commented out, **so please look it over before continuing. One would want to un-comment the whole section at the bottom of the `setup.py`.** One would also like to add a `setup.cfg` e.g.:

```config
[flake8]
ignore = E121,E123,E226,W292
max-line-length = 160
```

#### Excluding depenencies

If there are issues with dependencies in your project e.g. : https://github.com/linkedin/pygradle/issues/96 one could exclude the dependency as follows:

```groovy
dependencies {
    python ('pypi:connexion:1.0.129') {
        exclude module: 'functools32'
    }
}
```

### Building and Running Tasks

There are two ways to run the tasks and build either with `gradle` or `./gradlew` commands. In the working directory where `build.gradle` file is available one can make use of the following commands.

Building an running tasks is the same as the Gradle, however here are some examples customised to PyGradle:

Tasks:
* clean: `gradle clean`
* build: `gradle build`
* other tasks: `gradle :runTests` (see below how to add this task) or `gradle pytest`
* exclude tasks: `gradle build -x pytest`
* do all: `gradle clean build -x pytest`
* see tasks: `gradle tasks --all` and dependencies `gradle dependencies` (previous to Gradle 3.+ all the tasks could be seen simply with `gradle tasks` with 3.+ the `--all` parameter was added to limit the number of tasks shown)
* see test coverage `gradle pytest coverage` it will generate a html report in `build/coverage/htmlcov`

In order to generate a HTML test report one could use `pytest-html` dependency and creating a Gradle task for that:
```groovy
// generate Tests report via a script.
task runTests(type:Exec) {
  workingDir '.'
  // Create the output folder
  doFirst {
    mkdir('build/test-report')
  }

  commandLine './runTests.sh'
}
```
where `runTests.sh` contains the command to activate the virtualenv created by PyGradle and run the tests.

```bash
#! /bin/bash
command -v source >/dev/null 2>&1 || {
  echo "I require source but it's not installed.  Aborting." >&2; exit 1;
}
source activate
py.test tests --html=build/test-report/index.html --self-contained-html
```

### Example PyGradle Projects

Besides the examples provided https://github.com/linkedin/pygradle/tree/master/examples we provide two additional ones:
- Example of of building a Falcon REST API using PyGradle (using python-web-app plugin) https://github.com/blankdots/pygradle-falcon-api
- Command line utility using [Click](http://click.pocoo.org/6/) for converting RDF to JSON using a [JSON-LD frame](http://json-ld.org/spec/latest/json-ld-framing/) https://github.com/blankdots/pygradle-jsonld-cli

## Documentation

Getting started:
* [com.linkedin.python Gradle Plugin](https://github.com/linkedin/pygradle/blob/master/docs/plugins/python.md)
* video
  * Gradle: Novice to Master @LinkedIn - Gradle Summit 2016 - https://www.youtube.com/watch?v=B79g5YuU_2Y
* source code
  * https://github.com/ethankhall/gradle-summit-novice-to-master

Little as there is the documentation for PyGradle is somewhat useful:
* videos
  * GradleSummit2015 Python builds with Gradle - https://www.youtube.com/watch?v=NRZAIcTA0N0
  * PyGradle @ LinkedIn - Gradle Summit 2016 - https://www.youtube.com/watch?v=kZWLcHtHx2U
  * Python Build Automation with Gradle at Linkedin - https://www.youtube.com/watch?v=At5X0bamEFI
* source code
  * Gradle examples for different code and IDEs: https://github.com/gradle/gradle/tree/f001402c8b7f404c923d972f1bc524b2de7b3fcd/subprojects/docs/src/samples
  * PyGradle example project https://github.com/linkedin/pygradle/tree/master/examples/example-project
