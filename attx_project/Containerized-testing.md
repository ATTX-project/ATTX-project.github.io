# Containerised Testing

This page describes the necessary steps required to run integration/BDD tests within a container or local environment alongside any required services running in their own containers.

<!-- TOC START min:1 max:3 link:true update:false -->
  - [Testing Workflow](#testing-workflow)
  - [Testing with Gradle](#testing-with-gradle)
  - [Structure of feature-test images](#structure-of-feature-test-images)
    - [runTests.sh script](#runtestssh-script)
    - [Gradle build files](#gradle-build-files)
    - [Dockerfile](#dockerfile)
    - [Configuring Jenkins](#configuring-jenkins)
  - [Running tests against custom images](#running-tests-against-custom-images)
  - [Developing tests](#developing-tests)

<!-- TOC END -->

In ATTX github repositories, IT/BDD (tests from now on) should be located in separate `x-feature-tests` projects (e.g. `gc-feature-tests`).

## Testing Workflow

Overview of the steps performed during the test suite:
* Start all required Semantic Broker containers
* Wait for the services to come up
* Build test code
* Build test container (only during the containerised testing task)
* Start test container (only during the containerised testing task)
* Run tests
* Export reports to the host machine
* Stop all containers
* The build is Successful or Failed depending on the results form the test container

## Testing with Gradle

There are two tasks for running the tests:
* `runIntegTests` -  for running the test in the local environment with all the ports exposed, thus allowing for rapid development; It will also start the containers needed, by default they will expose ports;
* `runContainerTests` - for running tests in the CI environment or a closed test setup, and for this one needs the Gradle property `-PtestEnv=CI`. This task will run the tests inside a container on the same network as the other containers without the need of exposing all the ports.

Running the tests inside the container:
`gradle -PregistryURL=attx-dev:5000 -PsnapshotRepoURL=http://attx-dev:8081/repository/attx-releases -PtestEnv=CI clean runContainerTests`
`gradle -PregistryURL=attx-dev:5000 -PsnapshotRepoURL=http://attx-dev:8081 -Penv=dev -PtestEnv=CI clean runContainerTests` (for pd-feature-tests)

Run the test locally without containers and exposing the ports:
`gradle -PregistryURL=attx-dev:5000 -PsnapshotRepoURL=http://attx-dev:8081/repository/attx-releases clean runIntegTests`

`gradle -PregistryURL=attx-dev:5000  -PsnapshotRepoURL=http://attx-dev:8081 -Penv=dev clean runIntegTests` (for pd-feature-tests)

Test reports are exported to the folder configured in `copyReportFiles` task (default `$buildDir/reports/`).

## Structure of Feature-test Images

### runTests.sh Script

This script is the starting point for a `testContainer`. It first checks and waits (for a limited time) for certain ports in the network to become available and then runs the test suite.
**TO DO**
In the future, the goal is to use Service discovery component to query for up-and-running services.

Example: Waiting for services to be available:
```bash
dockerize -wait tcp://mysql:3306 -timeout 240s
dockerize -wait http://fuseki:3030 -timeout 60s
dockerize -wait http://gmapi:4302/health -timeout 60s
```

[Dockerize (Github)](https://github.com/jwilder/dockerize)

These checks and waits are complementary to the ones done in the Gradle script and the reason is that some tests need to have some fixture in place (e.g. ATTX DPU) which also wait for other services to start. Thus the tests need to have everything ready before starting, meaning the last dependent container needs to finish before running.

If the time-out is reached and the service is still not available, the process exits with status code 1.

### Gradle Build Files

Test project contains two Gradle configurations, one for developing tests and managing containers and the other one for running tests inside the container; these are `build.gradle` and `build.test.gradle` respectively.

**build.gradle**

Example of the most relevant parts:

```groovy
if (!project.hasProperty("testEnv") || project.testEnv == "dev") {
    ext.testSet = "localhost"
} else if (project.testEnv == "CI"){
    ext.testSet = "container"
} else {
    throw new GradleException("Build project environment option not recognised.")
}

ext {
    testImageWF = "latest"
    testImageGM = "latest"
    testImageES5 = "latest"
}

dcompose {
    createComposeFile.useTags = true
    registry ("$registryURL") {
        // no user/pass
    }
    networks {
      gcTest
    }

    es5 {
        forcePull = true
        forceRemoveImage = true
        image = "${imageRepo}:${imageRepoPort}/attx-es5:${testImageES5}"
        networks = [gcTest]
        hostName = 'es5'
        if (testSet == "localhost") {
            portBindings = ['9210:9210', '9310:9310']
        }
    }

    wfapi {
        forcePull = true
        forceRemoveImage = true
        image = "${imageRepo}:${imageRepoPort}/wf-api:${testImageWF}"
        dependsOn = [mysql]
        networks = [gcTest]
        hostName = 'wfapi'
        if (testSet == "localhost") {
            portBindings = ['4301:4301']
        }
    }

    gmapi {
        forcePull = true
        forceRemoveImage = true
        image = "${imageRepo}:${imageRepoPort}/gm-api:${testImageGM}"
        dependsOn = [wfapi, es5]
        networks = [gcTest]
        hostName = 'gmapi'
        if (testSet == "localhost") {
            portBindings = ['4302:4302']
        }
    }

    test {
        ignoreExitCode = true
        baseDir = file('.')
        dockerFilename = 'Dockerfile'
        if (testSet == "container") {
            binds = ["/var/run/docker.sock:/run/docker.sock"]
        }
        dependsOn = [gmapi] // This dependency is not really needed however it reminds the scope of the tests.
        command = ['sh', '-c', '/tmp/runTests.sh']
        waitForCommand = true
        forceRemoveImage = true
        attachStdout = true // to view the output in the console
        networks = [gcTest]
    }
}

task copyReportFiles(type: DcomposeCopyFileFromContainerTask) {
    service = dcompose.test
    containerPath = '/tmp/build/reports/tests'
    destinationDir = file("build/reports/")
    cleanDestinationDir = false
}

startTestContainer.finalizedBy copyReportFiles

shadowJar {
    classifier = 'tests'
    from sourceSets.test.output
    configurations = [project.configurations.testRuntime]
}

// making sure the that fresh build of test classes is done before building the image
// Other prerequisites for the tests (e.g. artifacts) should be added here.
buildTestImage.dependsOn shadowJar
buildTestImage.dependsOn testClasses

task checkDPUDone(type: DockerWaitContainer) {
    dependsOn startGmapiContainer
    targetContainerId {dcompose.attxdpus.containerId}
    doLast{
        if(getExitCode() != 0) {
            println "ATTX DPU Container failed with exit code \${getExitCode()}"
        } else {
            println "Everything is peachy."
        }
    }
}

startTestContainer.dependsOn checkDPUDone

task runContainerTests {
    dependsOn startTestContainer
    finalizedBy removeImages
    doLast {
        if(dcompose.test.exitCode != 0){ throw new GradleException("Tests within the container Failed!") }
    }
}
```

`dcompose` task configuration is first used to reference all the images required by the tests. In the example, test will require only the container (e.g. `gmapi`) that is the object of the tests, meanwhile the container that is the object of the tests will require the additional containers (`fuseki`, `essiren` and `gmapi`) in addition to the test container.
Service configurations should include at least `image` and in order to run the tests in a local environment the `portBindings` properties. Test container configuration is mostly same for all the test projects.

Rest of the configuration is the same for all the test projects (assuming that test container is always called `test`). `copyReportFiles` task copies report files from inside the container to the build/from-container directory.
`ShadowJar` task create one big jar with all the dependencies required to run the tests, which allows us to run Gradle within in the container in offline mode, which does not trigger dependency downloads (faster). Other option would have been to attach a volume with the preloaded dependencies, but that would also required maintenance.   

`checkDPUDone` waits for the ATTX DPUS to be added to the front-end where there is a need for such test fixtures. The ATTX DPU waits for MySQL  (for about 4 minutes to be up) in order to add the DPU. If everything is OK the exit code of that container will be (By convention, an 'exit 0' indicates success - http://www.tldp.org/LDP/abs/html/exit-status.html). The same strategy is used to check the Tests fail or not inside the container - if the exit code is `0` the tests were successful if not the tests failed.

**build.test.gradle**

This is a very minimal build file that basically only includes dependency configuration.

The system properties are required to be set in the container and these need to be the same as the `hostName` of the containers so that are no conflicts when accessing them in the network. The solution of having them inside the Gradle configurations instead of the test code is a decision of keeping the implementation independent of the configuration of the containers.

Another reason is to allow for the tests to be run in local environment, meaning the ports are exposed and the tests need to take into consideration that the requests make inside the network between containers need to be done under the `hostName` of the specific containers (e.g. no http://localhost:4301/health but instead http://wfapi:4301/health).

```groovy
plugins {
    id "java"
}

dependencies {
    testCompile fileTree(dir: 'build/libs', include: ['*.jar'])
}

task integTest(type: Test){
    doFirst {
        systemProperty 'frontend.port', 8080
        systemProperty 'frontend.host', 'frontend'
        systemProperty 'wfapi.port', 4301
        systemProperty 'wfapi.host', 'wfapi'
        systemProperty 'fuseki.port', 3030
        systemProperty 'fuseki.host', 'fuseki'
        systemProperty 'gmapi.port', 4302
        systemProperty 'gmapi.host', 'gmapi'
        systemProperty 'essiren.port', 9200
        systemProperty 'essiren.tcp', 9300
        systemProperty 'essiren.host', 'essiren'
        systemProperty 'es5.port', 9210
        systemProperty 'es5.host', 'es5'
    }
    forkEvery = 2
    maxParallelForks = 2
    testLogging {
        showStackTraces = true
    }
    doLast {
        systemProperties.remove 'frontend.port'
        systemProperties.remove 'frontend.host'
        systemProperties.remove 'wfapi.port'
        systemProperties.remove 'wfapi.host'
        systemProperties.remove 'fuseki.port'
        systemProperties.remove 'fuseki.host'
        systemProperties.remove 'gmapi.port'
        systemProperties.remove 'gmapi.host'
        systemProperties.remove 'essiren.port'
        systemProperties.remove 'essiren.tcp'
        systemProperties.remove 'essiren.host'
        systemProperties.remove 'es5.port'
        systemProperties.remove 'es5.host'
    }
}
```

Dependencies are loaded from the überjar created by the `shadowJar` task in the `build.gradle`. This jar and classes directory includes everything that is needed to run the tests, so we can actually run Gradle in the offline mode (`--offline`). Building step is faster, since we don't have to download all the dependencies every time the tests are run.

### Dockerfile Description

```Dockerfile
FROM frekele/gradle:3.3-jdk8

RUN apt-get update \
    && apt-get install -y wget \
    && apt-get clean

ENV DOCKERIZE_VERSION v0.3.0

RUN wget https://github.com/jwilder/dockerize/releases/download/$DOCKERIZE_VERSION/dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && tar -C /usr/local/bin -xzvf dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz \
    && rm dockerize-linux-amd64-$DOCKERIZE_VERSION.tar.gz

RUN mkdir -p /tmp/

WORKDIR /tmp/

COPY build /tmp/build
COPY build.test.gradle /tmp/build.gradle
COPY runTests.sh /tmp/
RUN chmod 700 /tmp/runTests.sh
```

`frekele/gradle` image contains all the components to run Gradle 3.3. `dockerize` is used by the `runTests.sh` script to poll ports on different containers as a crude way to determine whether the service each container provides is up or not. Note that only `build.test.gradle` file is copied to the image. Image also doesn't run any script by default, instead the dcompose gradle plugin is used to attach a command  `runTests.sh` to the test container, which runs dockerized checks and runs the tests.

### Configuring Jenkins

Example configuration for Jenkins using pipeline plugin and declarative configurations:

```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') { // for display purposes
            steps {
                // Get some code from a GitHub repository
                git branch: 'dev', url: 'https://github.com/ATTX-project/platform-deployment.git'
            }
        }
        stage('Compile/Package/Test') {
            steps {
                echo sh (script: "${GRADLE_HOME}/bin/gradle --console=plain -b ${workspace}/build.gradle -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://archiva:8080 -PtestEnv=CI :pd-feature-tests:clean :pd-feature-tests:runContainerTests", returnStdout: true)
            }
            post {
                always {
                    echo 'publishing reports'
                    publishHTML target: [
                        allowMissing: false,
                        alwaysLinkToLastBuild: false,
                        keepAll: false,
                        reportDir: 'pd-feature-tests/build/reports/tests/integTest',
                        reportFiles: 'index.html',
                        reportName: 'pd-feature-tests-reports'
                        ]                    
                }
            }            
        }
    }     
}
```

Jenkins sets `testEnv=CI` in order to run the tests in a container. Report publication is configured as a `post` block of the build stage, so that is always run even if the build/test step fails.

## Running Tests Against Custom Images

If you want to run tests against other that 'latest' tag/versions of the images, do the following, where `testtag` is the tag/version needed for tests:

* Push your custom images to the private Docker repo with some tags (e.g. `attx-dev:5000/gm-api:testtag`).
* Modify `dcompose` configuration int he `build.gradle` of the project you are working with by changing the tag of the selected images. For example:

```groovy
dcompose {
...
  gmapi {
    image = 'attx-dev:5000/gm-api:testtag'
  }
...
}
```

## Developing Tests

When developing tests, it is easier to just run them inside the IDE and work on services on the `localhost` or use the `runIntegTests` tasks for running the integration tests with the containers exposing ports.
