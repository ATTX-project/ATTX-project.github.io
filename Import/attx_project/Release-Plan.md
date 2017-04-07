Adjacent to the - [Feature roadmap](https://github.com/ATTX-project/project-management/wiki/Feature-roadmap) the release plan is detailed below, alongside corresponding issues.

**We should establish priority and complexity for the issues above in order to have a complete release plan.**

**We should associate issues with the first release version.**

<!-- TOC START min:1 max:3 link:true update:true -->
  - [Realease Version 2.0](#realease-version-20)
  - [Realease Version 1.0](#realease-version-10)

<!-- TOC END -->

## Realease Version 2.0
Planned release date: **30st of April 2017**

## Realease Version 1.0

Planned release date: **31st of March 2017**

1. Exposed ports (NEED TO CHECK - might be solved; very small task)
    - DC component should have the ports exposed for both instances of ElasticSearch
    (stick with one version of ES ?) 9200 & 9300 + 9210 & 9310
    - UV should have a frontend that expose port 8080
    - other ports should not be available, except for the test/dev environments; this might require changes to the Gradle scripts (scripts out of scope for first release)
2. Versioning of artifacts and images
    - _Make sure that packages in java projects follow the same structure (group and artifact)_ https://github.com/ATTX-project/project-management/issues/86
    - _Make sure that all the artifacts end up in Archiva_ https://github.com/ATTX-project/project-management/issues/81
    - _Replace downloads from Jenkins with Archiva downloads_ https://github.com/ATTX-project/project-management/issues/80
    - _Publishing release artifacts externally_ https://github.com/ATTX-project/platform-deployment/issues/25
    - _Deployment of release images to a server_ https://github.com/ATTX-project/platform-deployment/issues/22
        - Images on the docker hub and on the private registry should have the same name and different tags for test/dev/prod ? maybe a pipeline can produce multiple tags
        - (out of scope for first release ?) Investigate in Gradle
        - investigate and set up CI which can work with Github other than internal Jenkins ? - needed for setting up a continuously release environment and publishing artifacts
    - _Generate release artifacts_ https://github.com/ATTX-project/platform-deployment/issues/21
       - artifacts should be published to Github and tagged with versions
       - tagging of artifacts is not uniform (not an issue) however might confuse users
    - _Nicer way to download required artifacts_ https://github.com/ATTX-project/platform-deployment/issues/24
3. Distribution of components
    - _Services Health check endpoint_ https://github.com/ATTX-project/platform-deployment/issues/4 (out of scope)
    - services discovery (out of scope for first release)
    - using IAAS platform https://github.com/ATTX-project/platform-deployment/issues/26
    - Create Docker Swarm out of ATTX containerised stack https://github.com/ATTX-project/project-management/issues/46
        - deployment scripts: Ansible ? Docker Swarm ?
4. missing functionality (MISSING tasks)
    - GM-API missing linking functionality https://github.com/ATTX-project/graph-component/issues/17
    - GM-API test clustering functionality with URLs that do not have graphs materialised in the graph store
    - UV missing DPU for scheduling GM-API; the DPUs need to be synchronised in order to propagate data in throughout the components
5. missing tests
    - _Update BDD tests to run in the new containerised environment_ https://github.com/ATTX-project/project-management/issues/72
        - dc-feature-tests missing
        - gc-feature-tests extended for other functionalities
        - wf-feature-tests not working
    - _Get rid of dpu jars in the pd-feature-tests project_ https://github.com/ATTX-project/platform-deployment/issues/23
        - pd-feature-tests not working
    - missing BDD tests for GC, WF, PD https://github.com/ATTX-project/project-management/issues/82
    - _Add automated user story tests for use case 1_ https://github.com/ATTX-project/platform-deployment/issues/17
6. missing documentation
    - installation instructions HIGH IMPORTANCE
    - development instructions
    - missing documentation for each component how to run them in standalone or how to integrate them within other processes
7. logging (out of scope for first release)
    - _Log collection from all components in order to check status and possible errors_ https://github.com/ATTX-project/distribution-component/issues/7
        - ELK stash available (latest version) - NOTE: maintaining 2 different versions of ES would be difficult
        - standardise logging format in components in order to have them available in logstash
        - (out of scope for first release) figure out a way to connect it with provenance data
8. Set up continuously release environment (HIGH IMPORTANCE)
    - _Plan and document release flow_ https://github.com/ATTX-project/platform-deployment/issues/20
    - _Setup continuous release environment_ https://github.com/ATTX-project/platform-deployment/issues/19
        - Jenkins and Archiva to release artifacts
        - set up github and private jenkins to release artifacts (out of scope)
        - set up jenkins and docker hub to release images with different tags and the same with private the repository
        - test before release - if unit/integration/BDD tests fail the artifacts cannot be deployed and this needs to be signalled. (out of scope)
