# UnifiedViews Plugins

Github repository: https://github.com/ATTX-project/uv-plugins

New Custom Unified Views plugins developed:
* `uv-common`
* `attx-l-replaceds`
* `attx-t-describeds`
* `attx-t-describeexternalds`
* `attx-t-rmlservice`


Old Custom Unified Views plugins developed:
* `uv-t-attx-template` plugin - basic template for creating a UnifiedViews plugin;
* `uv-t-attx-metadata` plugin - collecting metadata specific to the input and output graphs;
* `uv-dpu-l-attx-apicaller` plugin - general functionality for calling an API and retrieving data;
* `uv-dpu-l-attx-esindexer` plugin - indexing data at a specific ElasticSearch search endpoint
* `uv-dpu-l-attx-linker` plugin - linking data based on [Linking Strategies](Linking-Strategies.md);
* `uv-dpu-l-attx-uploader` pluging - uploading data;
* `uv-dpu-t-attx-uc1-infras2internal` plugin - specific to [UC1 - Infrastructure and Publications](Use-case-Infrastructures-and-publications.md).

All the plugins play a key role in the user interface of the ATTX Semantic Broker as exemplified in [Data Administrator's User guide](User-Guide-Administrator.md).

## Creating New Plugins

https://grips.semantic-web.at/display/UDDOC/Tutorial%3A+Creating+new+DPU

These instructions create a Maven project. We can integrate it with Gradle using a Maven plugin. In `build.gradle` file configuration would look like:

```groovy
plugins {
    id "com.github.dkorotych.gradle-maven-exec" version "1.0"
}

task repackage(type: MavenExec) {
    goals 'clean', 'package'
}
```
