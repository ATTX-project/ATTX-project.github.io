<!-- TOC START min:1 max:3 link:true update:true -->
  - [ATTX-DEV SERVER](#attx-dev-server)
  - [How to access the Jenkins server](#how-to-access-the-jenkins-server)
  - [ATTX-SANDBOX SERVER](#attx-sandbox-server)

<!-- TOC END -->

## ATTX-DEV SERVER

Sudoers: jkesanie, madeira, stefanne

The ATTX-DEV server (`attx-dev.hulib.helsinki.fi`) runs the following containerised services: Jenkins (automated Gradle tests and Docker building), Docker Registry Server (private registry for ATTX-Project Docker images), and NGINX (web server).

All services are implemented as Docker containers, and the goal is to have their images publicly available in Docker Hub (cf. https://hub.docker.com/u/attxproject/dashboard/) for the general public's benefit.

To ensure the availability of the CI/CD environment, the Jenkins and Docker Registry containers have been put under the supervision of the "sysctl" daemon ("jenkins-docker.service" and "private-docker-registry.service", respectively). This will ensure their automatic start-up in case the attx-dev server needs to be rebooted for maintenance.

_TODO: put NGINX under sysctl supervision._

## How to access the Jenkins server
Please request an account to the ATTX Project team. The Jenkins WebUI can be reached @ [http://attx-dev:49001/](http://attx-dev:49001/)


## ATTX-SANDBOX SERVER

Sudoers: jkesanie, madeira, stefanne

The ATTX-SANDBOX server (attx-sandbox.hulib.helsinki.fi) runs the containerised services described in the [Platform Deployment](Platform-Deployment.md).
