# Instructions for Generating ATTX Github.io

## Automatic build & deployment

Making use of: https://github.com/ATTX-project/gradle-pullrequest-builder

Configuring a Jenkins pipleline:
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') { // for display purposes
            steps {
                // Get some code from a GitHub repository
                git branch: 'dev', url: 'https://<user>:<user_token>@github.com/ATTX-project/ATTX-project.github.io.git'
            }
        }
        stage('Compile/Package/Test') {
            steps {
                echo sh (script: "${GRADLE_HOME}/bin/gradle --console=plain -b ${workspace}/build.gradle -PartifactRepoURL=http://archiva:8080 -Puser=<user> -Ptoken=<user_token> -PjenkinsBuild=${BUILD_NUMBER} clean gitPullRequest", returnStdout: true)
            }
        }
    }

}
```

## Local build & deployment

### Prerequisites

1. Install or download Node.js https://nodejs.org/, this will also install NPM.
2. Install Gitbook CLI https://github.com/GitbookIO/gitbook-cli `$ npm install -g gitbook-cli`
3. Verify installation `$ gitbook --version`

### Make changes and deploying HTML Page

Working directory for the Markdown files  is `attx_project`, change any of the Markdown files or add new ones, in the respective directory. **In order for the files to be visible in the table of contents, make changes to SUMMARY.md . The index is generated based on README.md**

Preview and serve your book using (in attx_project directory):

```
$ gitbook serve
```

Or build the static website using:

```
$ gitbook build
```

For other instructions check: https://github.com/GitbookIO/gitbook/blob/master/docs/setup.md

The output is generated in `attx_project/_book` folder.
**In order to deploy/update the ATTX Github.io copy the contents of the folder `attx_project/_book` to the root directory.**

If you're using ZSH, enter the command in the project root directory: `$ cp -r attx_project/_book/* .`

In case you're using BASH, enter the copy command as follows: `$ cp -r attx_project/_book/. .`

### Troubleshooting

When installing with `sudo` one might need to change ownership for the gitbook:
* `$ sudo chown -R $USER:$GROUP /usr/bin/gitbook`
* `$ sudo chown -R $USER:$GROUP /usr/lib/node_modules/gitbook-cli`
* `$ sudo chown -R $USER:$GROUP /home/$USER/.gitbook`
