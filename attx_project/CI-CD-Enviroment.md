# CI/CD Environment

<!-- TOC START min:1 max:3 link:true update:false -->
- [ATTX-DEV SERVER](#attx-dev-server)
- [How to access the Jenkins server](#how-to-access-the-jenkins-server)
- [How to access the Archiva server](#how-to-access-the-archiva-server)
- [How to access the PyPi Ivy repo server](#how-to-access-the-pypi-ivy-repo-server)
- [ATTX-SANDBOX SERVER](#attx-sandbox-server)

<!-- TOC END -->

**Currently all these services are located on servers with restricted access.**

## ATTX-DEV SERVER

The ATTX-DEV server (`attx-dev.hulib.helsinki.fi`) runs the following containerised services: Jenkins (automated Gradle tests and Docker building), Docker Registry Server (private registry for ATTX-Project Docker images), and Nginx (web server).

All services are implemented as Docker containers, and the goal is to have their images publicly available in Docker Hub (cf. https://hub.docker.com/u/attxproject/dashboard/) for the general public's benefit.

To ensure the availability of the CI/CD environment, the Jenkins and Docker Registry containers have been put under the supervision of the `sysctl` daemon (`jenkins-docker.service` and `private-docker-registry.service`, respectively). This will ensure their automatic start-up in case the `attx-dev` server needs to be rebooted for maintenance.

## How to Access the Jenkins Server
The Jenkins WebUI can be reached @ [http://attx-dev:49001/](http://attx-dev:49001/)
More information about [Jenkins CI](Jenkins-CI.md)

## How to Access the Archiva Server
The Archiva WebUI can be reached at: [http://attx-dev:8081/](http://attx-dev:8081/)

To access the REST API use:
* the documentation: http://attx-dev:8081/#rest-docs-archiva-rest-api/index.html
* to download an artifact (e.g. gmAPI artifact):
```config ${artifactRepoURL}/restServices/archivaServices/searchService/artifact?g=org.uh.hulib.attx.gc&a=gm-API&v=${gmAPI}&p=tar.gz":"gm-API-${gmAPI}.tar.gz
```

## How to Access the PyPi Ivy Repo Server

Example of curl requests:
* ```shell
curl -X GET -H http://attx-dev:5039/pypi/{dependencyName}/{version}/dependencyName-version.tar.gz
``` - to retrieve dependency with a specific version number
* ```shell
curl -X POST -H "Content-Type: application/json" -H "Cache-Control: no-cache" -d '{ "dependencies": [ { "name" : "pytest", "version" : "3.0.5" }, { "name" : "elizabeth", "version" : "0.3.11" } ], "replace" : [ { "name": "alabaster", "oldVersion": "0.7", "newVersion": "0.7.1" } ] }' "http://attx-dev:5639/add"
``` - for adding dependencies

## ATTX-SANDBOX SERVER

The ATTX-SANDBOX server (`attx-sandbox.hulib.helsinki.fi`) runs the containerised services described in the [Platform Deployment](Platform-Deployment.md).
