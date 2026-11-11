#!groovy

pipeline {

    agent none

    options {
        buildDiscarder(logRotator(daysToKeepStr: '21', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    triggers {
       ciBuildTrigger(
           noSquash: true,
           providerList: [
               rabbitMQSubscriber(
                   name: 'RabbitMQ',
                   overrides: [
                       topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
                       queue: 'osci-pipelines-queue-11'
                   ],
                   checks: [
                       [field: '$.update.release.dist_tag', expectedValue: '^(f[3-9]{1}[0-9]{1}|eln)$']
                   ]
               )
           ]
       )
    }

    parameters {
        string(name: 'CI_MESSAGE', defaultValue: '{}', description: 'CI Message')
    }

    stages {
        stage('Trigger Testing') {
            steps {
                script {
                    def msg = readJSON text: params.CI_MESSAGE

                    if (msg) {
                        def branch = msg['update']['release']['branch']
                        def bodhiId = msg['update']['updateid']
                        currentBuild.displayName = bodhiId

                        def allTaskIds = [] as Set

                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add(build['task_id'])
                        }

                        def artifactId
                        def additionalArtifactIds

                        def multihostPipeline = true
                        if (branch == 'f45' || branch == 'f44' || branch == 'f43') {
                            multihostPipeline = false
                        }

                        if (allTaskIds) {

                            def counter = 0
                            allTaskIds.each { taskId ->
                                artifactId = "koji-build:${taskId}"
                                additionalArtifactIds = allTaskIds.findAll{ it != taskId }.collect{ "koji-build:${it}" }.join(',')

                                build(
                                    job: "fedora-ci/dist-git-pipeline/master",
                                    wait: false,
                                    parameters: [
                                        string(name: 'ARTIFACT_ID', value: artifactId),
                                        string(name: 'ADDITIONAL_ARTIFACT_IDS', value: additionalArtifactIds),
                                        string(name: 'DIST_GIT_BRANCH', value: branch)
                                        booleanParam(name: 'MULTIHOST_PIPELINE', value: multihostPipeline)
                                    ]
                                )

                                // wait a bit before triggering testing for the next build;
                                // some updates can have dozens or hundreds of builds,
                                // and Jenkins could struggle to handle so many requests at the same time
                                if (counter > 1) {
                                    sleep(time:20, unit:"SECONDS")
                                }
                                counter += 1
                            }
                        }
                    }
                }
            }
        }
    }
}
