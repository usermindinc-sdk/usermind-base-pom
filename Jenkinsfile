#!/usr/bin/env groovy

node {
    try {

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

        stage('build') {
            // Let people know a build has begun
            if(env.BRANCH_NAME == 'develop' || env.BRANCH_NAME == 'master') {
                // Ensure that the application name is appropriate may need to include -application after artifactid
                if(slackMessageDestination != "@Jenkins") {
                    util.sendSlackMessage(slackMessageDestination, ":jenkins: ${pom.artifactId} ${pom.version} build started: <${env.BUILD_URL}|${env.JOB_NAME}#${env.BUILD_NUMBER}> \n ${changeLogMessage}")
                }
                // Ensure that the application name is appropriate may need to include -application after artifactid
                util.sendSlackMessage(slackTeamMessageDestination, ":jenkins: ${pom.artifactId} ${pom.version} build started: <${env.BUILD_URL}|${env.JOB_NAME}#${env.BUILD_NUMBER}> \n ${changeLogMessage}")
                // Add test related commands as appropriate eg -Dbasepom.test.timeout=0 -Dbasepom.failsafe.timeout=0
                sh 'mvn -B clean deploy'
            }
        }
    }

    catch (buildError) {
        currentBuild.result = 'FAILURE'
        throw buildError
    }

}

