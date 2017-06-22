# ATTX Development Environment and Guidelines

This page describes the project management and software development related practices and tools for the ATTX-2016 project. This is the place to go, when member of the project wants to learn or be reminded of how different project management methodologies and tools are used in ATTX, how we for example branch in using GIT, or what are the requirements for testing ones code.

## Table of Contents
<!-- TOC START min:1 max:3 link:true update:false -->
  - [Meetings](#meetings)
    - [Sprint Planning](#sprint-planning)
    - [Daily](#daily)
    - [Grooming](#grooming)
    - [Retro/Demo](#retrodemo)
  - [Github Workflow](#github-workflow)
    - [Practicalities](#practicalities)
    - [Issue Management](#issue-management)
    - [Privacy Considerations](#privacy-considerations)
    - [Source Code & Other Files](#source-code--other-files)
    - [Branching](#branching)
  - [Testing](#testing)
    - [Unit Testing](#unit-testing)
    - [BDD Testing](#bdd-testing)
    - [Test Coverage](#test-coverage)
  - [ZenHub](#zenhub)

<!-- TOC END -->

***

## Meetings

### Sprint Planning

Duration: 2 hours max
Monday morning when the sprint starts.

PO should be present at least at the end of the meeting.

### Daily

Duration: 15 min
Every day at 9:30.

Issues that are ready should be closed after the meeting and the burn-down chart should be updated to reflect the current situation.

### Grooming

Duration: 30-60 min
Middle of the sprint on Friday.

### Retro/Demo

Duration: 30 min
At the end of the project on Friday, right before lunch.
PO must be present. Scrumm master presents the sprint and demo.

## Github Workflow

Project uses Github for all of its issue management purposes from general todos (e.g. prepare materials for the next meeting)  to sprint related development task management. ATTX has been registered as organization in Github (https://github.com/ATTX-project) under which all the project related repositories will be created. Project is also subscribed to the Github's commercial plan, which allows the creation unlimited private repositories.
Project uses ZenHub (https://www.zenhub.com/) to add project management related functionality, such as Epics, todo-list and Broads to Github. One needs to have ZenHub installed in orde to participate in Sprint activities.

### Practicalities

* Add Milestone for every sprint and include the sprint issue in the corresponding milestone. This allows us to take (some) advantage of the timing capabilities related to milestones in Github/Zenhub combo.


### Issue Management

_Every task/user story is an issue_

* project-management repository is used for issues that do not fit under component specific repositories. The visibility of project-management repository is private, so that only project members are allowed to add new issues. The idea is that project-management repository is used for project's internal, "Buy more coffee" kind of issues.
* All the EPICs will be created in the project-management repository.
* Issues related to certain Sprint are collected to an EPIC called "Sprint X". Issues can be added to EPIC when they are created or when they are assigned.
* Milestones are not used in PM repo. In other repositories milestones can be used to group issues for releases.
* One should always reference the issues in the commit message. Use full name ATTX-project/[repo]/#[issue number] to distinguish between repositories.

### Privacy Considerations

**Do not include any personal information that is not part of the Github in the issues.**

### Source Code & Other Files

Project's GitHub is divided into following repositories:

* project-management
 * Special private repository main for providing overall view to the issues in different repositories.
* etl
 * Repository for projects that provide etl/workflow related functionality. Add new directories for every specific implementation.
* public-apis
* graph-component
 * Repository hosts a single project that manages the state of the internal graph.

All projects are Gradle projects. Naming convention for the projects is [root-project]-[subproject] e.g. workflow-component-feature-tests or workflow-component-wfAPI.

Unit tests can be written in any language.

BDD tests should be run with [Cucumber JVM implementation](https://cucumber.io/docs/reference/jvm)

Project structure:
```config
gradle root project (e.g. wf component)
│   README.md
│   build.gradle    
│   docker-compose.yml
└───subprojectX (e.g. Unified Views implementation of WF component)
│   └───unit tests
│   │   │   unit tests (including the ones for the libraries)
│   └───library
└───subprojectY (e.g. WFAPI implementation of Wf Component)
│    │   unit tests
└───componentX-feature-tests
│    │   cucumber feature files and implementation for each subproject
└───etc...
```

```config
platform-deployment
│   README.md
│   build.gradle  
│   all-docker-compose.yml
└───workflow-component
│   │   wfc-docker-compose.yml
│   │
│   └───WFAPI
│   │   │   Dockerfile
│   └───UnifiedViews
│       │   Dockerfile
└───distribution-component
│    │   dc-docker-compose.yml
│    └───ElasticSearch-Siren
│    │   │   Dockerfile
└───etc...
```

### Branching

**A must read**: Project uses the branching model described by Vincent Driessen (http://nvie.com/posts/a-successful-git-branching-model/). One can print it out (http://nvie.com/files/Git-branching-model.pdf) for reference.

## Testing

### Unit Testing

Framework to use for unit testing depends on the programming language used.
List of unit testing frameworks:
* Java - [JUnit](http://junit.org/)
* Python - [Unittest](https://docs.python.org/3/library/unittest.html) & [nose](http://nose.readthedocs.io/en/latest/index.html)
* Javascript - [Mocha](https://mochajs.org/) & [Chai](http://chaijs.com/)

For unit tests, use which ever framework one feel comfortable.

### BDD Testing

Feature files are written using Gherkin language (https://github.com/cucumber/cucumber/wiki/gherkin).

Tests are executed using JVM version of the Cucumber, which allows for writing tests in any JVM supported language. Currently the aim is to support BDD testing in Java and Python.

#### Setup Netbeans for BDD Testing

Plugins
* [Gradle](https://gradle.org/)
* [Gherkin](https://github.com/cucumber/cucumber/wiki/Gherkin)

By default, Gradle plugin seems to be using 2.31 version of the the Gradle. One must upgrade this to 3.1. by going to the project properties and changing the Gradle home.

### Test Coverage

**Java**

Use [JaCoCo](http://www.eclemma.org/jacoco/) (still in incubation)
Documentation for the JaCoCo Gradle plugin: https://docs.gradle.org/current/userguide/jacoco_plugin.html

## ZenHub

[ZenHub](https://www.zenhub.com/)

When moving an Epic issue from one repository to another, the EPIC status is not preserved.

ZenHub supports (since 6.1.2017) multi-repo milestones, which allows us to use both burndown and velocity charts.

**NOTE**
Closing a milestone must be done separately for every repository.
