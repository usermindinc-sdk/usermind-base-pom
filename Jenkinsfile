#!/usr/bin/env groovy

node {
    try {

        if (env.BRANCH_NAME != 'master') {
            //Only build the master branch. The others just don't matter.
            echo 'Jenkinsfile is set to only build the master branch.'
            currentBuild.result = "NOT_BUILT"
            error('Not building as this is not the master branch.')
        }

        stage('checkout') {
            deleteDir()
            checkout scm
            changeLogMessage = util.changeLogs()
        }

        stage('Configure environment') {
            build_config = util.loadJenkinsConfiguration("jenkins.yaml")
            util.useJDKVersion(build_config.javaVersion)
            util.useMavenVersion(build_config.mavenVersion)
            pom = readMavenPom file: 'pom.xml'

            // For you/your team to do: Choose a slack channel. For example, Skylab has a slack channel just for builds. If you just want the messages
            // to go to the author of the latest git commit, leave this as is (and delete the if block).
            // Remember that you need '@' (for direct messages) or '#' (for channels) on the front of the slackMessageDestination value.
            slackMessageDestination = "@${util.committerSlackName()}"
            // More complex example:
            if(util.isPullRequest() || env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'master') {
                // Change out for the appropriate team channel
                slackTeamMessageDestination = "#integration-build"
            }
            gitCommit = util.commitSha()
        }


        // configFileProvider([configFile(fileId: 'global-maven-config', targetLocation: '/var/lib/jenkins/.m2/settings.xml')])
        // {
        //     stage('Set version number and validate pom')
        //     {
        //         //Set the version based on the build number
        //         sh "mvn -DskipITs -U -B validate versions:set"
        //
        //         //check for released versions of snapshots and use them instead of snapshots. Fail if there aren't released versions.
        //         sh "mvn -DskipITs -U -B validate -DallowSnapshots=false -DfailIfNotReplaced=true versions:use-releases"
        //
        //        //Don't call a release build "SNAPSHOT"
        //         sh "mvn -DskipITs -U -B validate versions:set -DremoveSnapshot"
        //
        //         //Remove the backup pom made by the above command
        //         sh "mvn versions:commit"
        //         echo "Updated version for release build"
        //
        //         pom = readMavenPom file: 'pom.xml'
        //         pomVersion = pom.version
        //         artifactId = pom.artifactId
        //         echo "pom version is set to $pomVersion"
        //     }
        // }

        stage('build') {
            // Let people know a build has begun
            if(env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'master') {
                // Ensure that the application name is appropriate may need to include -application after artifactid
                if(slackMessageDestination != "@Jenkins") {
                    util.sendSlackMessage(slackMessageDestination, ":jenkins: ${pom.artifactId} ${pom.version} build started: <${env.BUILD_URL}|${env.JOB_NAME}#${env.BUILD_NUMBER}> \n ${changeLogMessage}")
                }
                // Ensure that the application name is appropriate may need to include -application after artifactid
                util.sendSlackMessage(slackTeamMessageDestination, ":jenkins: ${pom.artifactId} ${pom.version} build started: <${env.BUILD_URL}|${env.JOB_NAME}#${env.BUILD_NUMBER}> \n ${changeLogMessage}")
                // Add test related commands ass appropriate eg -Dbasepom.test.timeout=0 -Dbasepom.failsafe.timeout=0
                sh "mvn -DskipITs -U -B validate versions:set"
                sh 'mvn clean deploy -Ddockerfile.skip=true'
            }
        }

    }

    catch (buildError) {
        currentBuild.result = 'FAILURE'
        util.sendSlackMessage(slackMessageDestination, ":jenkins_rage: ${pom.artifactId} ${pom.version} build FAILED: ${env.BUILD_URL}consoleFull", "danger")
        util.sendFailureEmail(util.commitAuthorEmail())
        throw buildError
    }

}

//Temporary until this is accepted to the main jenkins util file
@NonCPS
def commitInfo(commit) {
    return commit != null ? "`${commit.commitId.take(7)}`  *${commit.msg}*  _by ${commit.author}_ \n" : ""
}

@NonCPS
def changeLogs() {
    String changeLog = ""
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            changeLog = changeLog + "${commitInfo(entry)}"
        }
    }

    if (changeLog) {
        changeLog = "Changes on *${env.BRANCH_NAME}* branch detected\n" + changeLog
    } else {
        commitHash = sh(returnStdout: true, script: 'git rev-parse HEAD').trim().take(7)
        commitText = sh(returnStdout: true, script: 'git show -s --format=format:"*%s*  _by %an_" HEAD').trim()
        changeLog = "No changes on *${env.BRANCH_NAME}* branch detected\n"
        changeLog = changeLog + "Building for commit: \n`${commitHash}` ${commitText}"
    }

    return changeLog
}
