
# Working with build Environments


<!-- TOC START min:1 max:3 link:true update:true -->
  - [Working with build Environments](#working-with-build-environments)
    - [Steps for Generating Release Version](#steps-for-generating-release-version)
    - [Example: Creating gmAPI test image for development](#example-creating-gmapi-test-image-for-development)
  - [References](#references)

<!-- TOC END -->

Each artifact (.tar.gz, .jar etc.) associated with the platform has a specific version associated (e.g. `0.1`, `1.0`, `1.0-SNAPSHOT` etc.), and these versions are set up from either the Gradle gradle.build script associated with each project or Maven specific `pom.xml` file. The only option at the moment to change the version of the built artifacts is to change them in their Gradle or Maven specific build files.

These artifacts (with associated version) are used to build Docker Images in the https://github.com/ATTX-project/platform-deployment/ repository. If the Docker Image allows it one can set the version of the artifact as environment variables (e.g. `gmAPI` and `rdfINDEXER`):

```{groovy}
gmapi {
    baseDir = file('docker')
    repository = "${imageBase}/gm-api:${imageGM}"
    portBindings = ['4302:4302']
    env = ["GMAPI=$gmAPI", "RDFINDEXER=$rdfINDEXER"]
  }
```

These are used in the generation of the Docker Images with specific tags depending on the environment as exemplified by the snippet below:

```{groovy}
if (!project.hasProperty("env") || project.env == "dev") {

    ext.wfAPI               = "0.1"
    ext.gmAPI               = "0.1"
    ext.rdfINDEXER          = "99"
    ext.attxMetadataPlugin  = "1.0-SNAPSHOT"
    ext.attxInfras2IPlugin  = "1.0-SNAPSHOT"
    ext.attxUploaderPlugin  = "1.0-SNAPSHOT"
    ext.attxAPIPlugin       = "1.0-SNAPSHOT"
    ext.imageES5            = "test"
    ext.imageWF             = "test"
    ext.imageGM             = "test"
    ext.imageATTXAPIPlugin  = "test"
    ext.imageATTXDPUs       = "test"
    ext.imageATTXShared     = "test"
    ext.imageFuseki         = "test"

} else if (project.env == "release"){

    ext.wfAPI               = "1.0"
    ext.gmAPI               = "1.0"
    ext.rdfINDEXER          = "1.0-SNAPSHOT"
    ext.attxMetadataPlugin  = "1.0-SNAPSHOT"
    ext.attxInfras2IPlugin  = "1.0-SNAPSHOT"
    ext.attxUploaderPlugin  = "1.0-SNAPSHOT"
    ext.attxAPIPlugin       = "1.0-SNAPSHOT"
    ext.imageES5            = "latest"
    ext.imageWF             = "latest"
    ext.imageGM             = "latest"
    ext.imageATTXAPIPlugin  = "latest"
    ext.imageATTXDPUs       = "latest"
    ext.imageATTXShared     = "latest"
    ext.imageFuseki         = "latest"

} else {
    throw new GradleException("Build project environment option not recognised.")
}
```

When running a Jenkins pipeline one can set the `env` property to `release` to build the realease version of the Docker Image e.g.:
```
gradle build -Penv=release -PjenkinsArtifactRepoURL=http://localhost:8080 -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://archiva:8080/repository/attx-releases clean :gm-API:buildGmapiImage
```

### Steps for Generating Release Version

Thus the steps for generating a release version are as follows:

1. Set up version of the artifact to build - this will trigger a Jenkins pipeline which will deploy the artifact to repository (e.g. Archiva, Jenkins, Github etc.)
1. Configure the Jenkins pipeline for building a Docker image to be the release version of that Docker Image, by specifying artifacts versions and image tags in the `common.gradle` from https://github.com/ATTX-project/platform-deployment/ repository
1. Publish artifacts to docker hub - TO BE done manually until https://github.com/ATTX-project/platform-deployment/issues/25 is done.
1. Create pull request for a repository from dev to master
1. Follow https://help.github.com/articles/creating-releases/ for creating a Github release

Alternative release generation:

1. Set up version of the artifact to build - this will trigger a Jenkins pipeline which will deploy the artifact to repository (e.g. Archiva, Jenkins, Github etc.)
1. Create pull request for a repository from a release-branch to master
1. (optional) Follow https://help.github.com/articles/creating-releases/ for creating a Github release
1. Configure the Jenkins pipeline for building a Docker image to be the release version of that Docker Image, by specifying artifacts versions and image tags in the `common.gradle` from https://github.com/ATTX-project/platform-deployment/ repository
1. Publish artifacts to docker hub - via Jenkins pipelines .


### Example: Creating gmAPI test image for development

I wanted to test out a new version of the gm-API artifact. In order to do that I have to first create and push a new version of the artifact and them build and push a new gmAPI image.

1. Commit and push new code to a branch `feature-X`. Create new version for the test artifact.
1. Create a new Jenkins pipeline using gm-api as the template
1. Configure new pipeline. Change brach to "feature-X" in the checkout stage.
1. Checkout platform-development code
1. Modify common.gradle and set ext.gmAPI to reference the test artifact and change the ext.imageGM tag to something else (e.g. test-featureX)
1. Build new test image:
```
gradle -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://attx-dev:8081 -Penv=dev buildGmapiImage
```
1. Push new test image:
```
gradle -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://attx-dev:8081 -Penv=dev pushGmapiImage
```

1. Push new release image:
```
gradle -PregistryURL=attx-dev:5000 -PartifactRepoURL=http://attx-dev:8081 -Penv=release -PdockerUser=account -PdockerPass=Test1234 pushGmapiImage
```

You might want to remove the test image when you are done with testing.

## References
* https://bitbucket.org/AlexanderBartash/gradle-environments-plugin/wiki/Home
* http://mrhaki.blogspot.fi/2009/11/gradle-goodness-using-properties-for.html
* https://github.com/marceloemanoel/gradle-environments-plugin
* https://github.com/nebula-plugins/nebula-release-plugin
