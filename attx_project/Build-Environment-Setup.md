# Build Environment Setup

### Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Working with Build Environments](#working-with-build-environments)
    - [Steps for Generating Release Version](#steps-for-generating-release-version)
    - [Example: Creating gmAPI test image for development](#example-creating-gmapi-test-image-for-development)
  - [References](#references)

<!-- TOC END -->

## Working with Build Environments

Each artifact (.tar.gz, .jar etc.) associated with the platform has a specific version associated (e.g. `0.1`, `1.0`, `1.0-SNAPSHOT` etc.), and these versions are set up from either the Gradle gradle.build script associated with each project or Maven specific `pom.xml` file. The only option at the moment to change the version of the built artifacts is to change them in their Gradle or Maven specific build files.

These artifacts (with associated version) are used to build Docker Images in the https://github.com/ATTX-project/platform-deployment/ repository. If the Docker Image allows it one can set the version of the artifact as environment variables (e.g. `gmAPI` and `rdfINDEXER`):

```groovy
gmapi {
    baseDir = file('docker')
    repository = "${imageBase}/gm-api:${imageGM}"
    portBindings = ['4302:4302']
    env = ["GMAPI=$gmAPI", "RDFINDEXER=$rdfINDEXER"]
  }
```

These are used in the generation of the Docker Images with specific tags depending on the environment as exemplified by the snippet below:

```groovy
if (!project.hasProperty("env") || project.env == "dev") {

    ext.uvProv                    = "2.0-SNAPSHOT"
    ext.gmAPI                     = "2.0-SNAPSHOT"
    ext.gmFrame                   = "2.0-SNAPSHOT"
    ext.indexing                  = "2.0-SNAPSHOT"
    ext.provenanceS               = "2.0-SNAPSHOT"
    ext.replacedsPlugin           = "1.0-SNAPSHOT"
    ext.describedsPlugin          = "1.0-SNAPSHOT"
    ext.describeexternaldsPlugin  = "1.0-SNAPSHOT"
    ext.rmlservicePlugin          = "1.0-SNAPSHOT"
    ext.sirenAPIPlugin            = "1.0-SNAPSHOT"
    ext.RMLService                = "0.0.1-SNAPSHOT"

    ext.imageBase           = "${imageRepo}:${imageRepoPort}"
    ext.imageFraming        = "${privateRepoTag}"
    ext.imageIndexing       = "${privateRepoTag}"
    ext.imageES5            = "${privateRepoTag}"
    ext.imageUVProv         = "${privateRepoTag}"
    ext.imageProvenance     = "${privateRepoTag}"
    ext.imageGM             = "${privateRepoTag}"
    ext.imageSirenAPIPlugin = "${privateRepoTag}"
    ext.imageATTXDPUs       = "${privateRepoTag}"
    ext.imageFuseki         = "${privateRepoTag}"
    ext.imageActivemq       = "${privateRepoTag}"
    ext.imageRMLService     = "${privateRepoTag}"

} else if (project.env == "release"){

    ext.uvProv                    = "2.0-SNAPSHOT"
    ext.gmAPI                     = "2.0-SNAPSHOT"
    ext.gmFrame                   = "2.0-SNAPSHOT"
    ext.indexing                  = "2.0-SNAPSHOT"
    ext.provenanceS               = "2.0-SNAPSHOT"
    ext.replacedsPlugin           = "1.0-SNAPSHOT"
    ext.describedsPlugin          = "1.0-SNAPSHOT"
    ext.describeexternaldsPlugin  = "1.0-SNAPSHOT"
    ext.rmlservicePlugin          = "1.0-SNAPSHOT"
    ext.sirenAPIPlugin            = "1.0-SNAPSHOT"
    ext.RMLService                = "0.0.1-SNAPSHOT"

    ext.imageBase           = "attxproject"
    ext.imageFraming        = "${releaseTag}"
    ext.imageIndexing       = "${releaseTag}"
    ext.imageES5            = "${releaseTag}"
    ext.imageUVProv         = "${releaseTag}"
    ext.imageProvenance     = "${releaseTag}"
    ext.imageGM             = "${releaseTag}"
    ext.imageSirenAPIPlugin = "${releaseTag}"
    ext.imageATTXDPUs       = "${releaseTag}"
    ext.imageFuseki         = "${releaseTag}"
    ext.imageActivemq       = "${releaseTag}"
    ext.imageRMLService     = "${releaseTag}"

} else if (project.env == "test"){

    ext.uvProv                    = "2.0-SNAPSHOT"
    ext.gmAPI                     = "2.0-SNAPSHOT"
    ext.gmFrame                   = "2.0-SNAPSHOT"
    ext.indexing                  = "2.0-SNAPSHOT"
    ext.provenanceS               = "2.0-SNAPSHOT"
    ext.replacedsPlugin           = "1.0-SNAPSHOT"
    ext.describedsPlugin          = "1.0-SNAPSHOT"
    ext.describeexternaldsPlugin  = "1.0-SNAPSHOT"
    ext.rmlservicePlugin          = "1.0-SNAPSHOT"
    ext.sirenAPIPlugin            = "1.0-SNAPSHOT"
    ext.RMLService                = "0.0.1-SNAPSHOT"

    ext.imageBase           = "attxproject"
    ext.imageFraming        = "${testTag}"
    ext.imageIndexing       = "${testTag}"
    ext.imageES5            = "${testTag}"
    ext.imageUVProv         = "${testTag}"
    ext.imageProvenance     = "${testTag}"
    ext.imageGM             = "${testTag}"
    ext.imageSirenAPIPlugin = "${testTag}"
    ext.imageATTXDPUs       = "${testTag}"
    ext.imageFuseki         = "${testTag}"
    ext.imageActivemq       = "${testTag}"
    ext.imageRMLService     = "${testTag}"

} else {
    throw new GradleException("Build project environment option not recognised.")
}
```

When running a Jenkins pipeline one can set the `env` property to `release` to build the release version of the Docker Image e.g.:

```shell
gradle build -Penv=release -PjenkinsArtifactRepoURL=http://localhost:8080 -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://archiva:8080/repository/attx-releases clean :gm-API:buildGmapiImage
```

### Steps for Generating Release Version

Thus the steps for generating a release version are as follows:

1. Set up version of the artifact to build - this will trigger a Jenkins pipeline which will deploy the artifact to repository (e.g. Archiva, Jenkins, Github etc.)
2. Configure the Jenkins pipeline for building a Docker image to be the release version of that Docker Image, by specifying artifacts versions and image tags in the `common.gradle` from https://github.com/ATTX-project/platform-deployment/ repository
3. Publish artifacts to docker hub, using Jenkins CI.
4. Create pull request for a repository from dev to master
5. Follow https://help.github.com/articles/creating-releases/ for creating a Github release

Alternative release generation:

1. Set up version of the artifact to build - this will trigger a Jenkins pipeline which will deploy the artifact to repository (e.g. Archiva, Jenkins, Github etc.)
2. Create pull request for a repository from a release-branch to master
3. (optional) Follow https://help.github.com/articles/creating-releases/ for creating a Github release
4. Configure the Jenkins pipeline for building a Docker image to be the release version of that Docker Image, by specifying artifacts versions and image tags in the `common.gradle` from https://github.com/ATTX-project/platform-deployment/ repository
5. Publish artifacts to docker hub - via Jenkins pipelines .


### Example: Creating Graph Manager Service test image for development

**Artifacts are also published to Docker hub with tag `dev`, along side the `latest` tag used for the private repository**

1. Commit and push new code to a branch `feature-X`. Create new version for the test artifact.
2. Create a new Jenkins pipeline using `gm-api` as the template
3. Configure new pipeline. Change branch to `feature-X` in the checkout stage.
4. Checkout platform-development code
5. Modify `common.gradle` and set `ext.gmAPI` to reference the test artifact and change the `ext.imageGM` tag to something else (e.g. `test-featureX`)
6. Build new test image:
```shell
gradle -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://attx-dev:8081 -Penv=dev :service-graphManager:buildGmapiImage
```
7. Push new test image:
```shell
gradle -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://attx-dev:8081 -Penv=dev :service-graphManager:pushGmapiImage
```
8. Push new release image:
```shell
gradle -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://attx-dev:8081 -Penv=release -PdockerUser=account -PdockerPass=Test1234 :service-graphManager:pushGmapiImage
```

9. Push to Docker hub `dev` image:
```shell
-PregistryURL=attx-dev:5000 -PartifactRepoURL=http://archiva:8080 -Penv=test -PdockerUser=account -PdockerPass=Test1234 :service-graphManager:pushGmapiImage
```

You might want to remove the test image when you are done with testing.

## References
* https://bitbucket.org/AlexanderBartash/gradle-environments-plugin/wiki/Home
* http://mrhaki.blogspot.fi/2009/11/gradle-goodness-using-properties-for.html
* https://github.com/marceloemanoel/gradle-environments-plugin
* https://github.com/nebula-plugins/nebula-release-plugin
