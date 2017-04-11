# Jenkins CI

The basic idea:

* Jenkins triggers, orchestrates, executes and monitors running of Gradle tasks using pipelines
* Gradle tasks are used to implement all the required actions related to pipelines (e.g. :buildDockerImage)

So instead of using Jenkins plugin to call Docker and create images, we should create a Gradle task that uses Docker and call that task from a Jenkins pipeline.

Orchestration is handled using Jenkins pipelines that all should share some of the common stages listed below.

![Figure 1. ATTX Platform Architecture](https://rawgit.com/ATTX-project/ATTX-project.github.io/master/images/cicd_jenkins-gradle-tools.svg)

Figure 1. Roles of Jenkins, Gradle and tools (i.e. Docker, java etc.)

<!-- TOC START min:1 max:3 link:true update:true -->
  - [Stages](#stages)
    - [Checkout](#checkout)
    - [Build/Test/Package](#buildtestpackage)
    - [Deploy artifacts](#deploy-artifacts)
    - [Publish reports](#publish-reports)
    - [Build image](#build-image)
- [Plugins](#plugins)
- [Additions to the Jenkins image](#additions-to-the-jenkins-image)

<!-- TOC END -->

## Stages

### Checkout

Getting the code from Github.

Example:

`git branch: 'dev', url: 'https://github.com/ATTX-project/distribution-component.git'`

### Build/Test/Package

Example:

`echo sh (script: "${GRADLE_HOME}/bin/gradle --console=plain -b ${workspace}/build.gradle :dc-elasticsearch-siren:attx-api-plugin:clean :dc-elasticsearch-siren:attx-api-plugin:build", returnStdout: true)`

We need both "echo" and "returnStdout: true" to get the output of the Gradle execution to Jenkins logs.

### Deploy artifacts

Copies output artifacts to somewhere where they can be used by other pipelines.

### Publish reports

Copies output of HTML reports to somewhere where Jenkins can access them.

Example:

`archiveArtifacts artifacts: 'dc-elasticsearch-siren/attx-api-plugin/build/libs/*.jar', onlyIfSuccessful: true`

Example: publishing test reports

```
        publishHTML([
            allowMissing: false,
            alwaysLinkToLastBuild: false,
            keepAll: false,
            reportDir: 'dc-elasticsearch-siren/attx-api-plugin/build/reports/tests/test',
            reportFiles: 'index.html',
            reportName: 'dc-elasticsearch-siren-test-reports'
            ])
```

### Build image

Artifacts are accessed from Jenkins. Normally artifacts are restricted to authenticated clients, but this can be changed by first setting property hudson.security.ArtifactsPermission = true (https://issues.jenkins-ci.org/browse/JENKINS-1871 and https://wiki.jenkins-ci.org/display/JENKINS/Features+controlled+by+system+properties) and then giving everyone access to the artifacts in the Jenkins security matrix.

Other option is to set "Allow anonymous read access" in the security settings.

# Plugins

* Join (https://wiki.jenkins-ci.org/display/JENKINS/Join+Plugin)
* Copy artifact (https://wiki.jenkins-ci.org/display/JENKINS/Copy+Artifact+Plugin)
* Artifact Deployer (https://wiki.jenkins-ci.org/display/JENKINS/ArtifactDeployer+Plugin)
* Green Balls (https://wiki.jenkins-ci.org/display/JENKINS/Green+Balls)
* JaCoCo (https://wiki.jenkins-ci.org/display/JENKINS/JaCoCo+Plugin)
* what else?

# Additions to the Jenkins image

Jenkins container must be linked to the pivy repository container:

`docker run -d --restart=always --link pivy_repo:pivyrepo  -v /var/run/docker.sock:/var/run/docker.sock -v $(which docker):/usr/bin/docker -p 49001:8080 -v /data/jenkins:/var/jenkins_home -t attxproject/attx-jenkins`

The following packages needs to added in order to build the wf-API python project:

* apt-get install libxml2-dev libxslt1-dev python-dev
* apt-get install build-essential
* apt-get install zlib1g-dev
