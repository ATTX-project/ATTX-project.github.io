# UnifiedViews Plugins

Custom Unified Views plugins developed:
* `uv-t-attx-template` plugin - basic template for creating a plugin
* `uv-t-attx-metadata` plugin - collecting metadata specific to the input and output graphs

## Creating new plugins

https://grips.semantic-web.at/display/UDDOC/Tutorial%3A+Creating+new+DPU

These instructions create a Maven project. We can integrate it with Gradle using a Maven plugin.

build.gradle
```groovy
plugins {
    id "com.github.dkorotych.gradle-maven-exec" version "1.0"
}

task repackage(type: MavenExec) {
    goals 'clean', 'package'
}
```
