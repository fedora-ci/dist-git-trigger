#!groovy


def msg
def artifactId
def additionalArtifactIds
def allTaskIds = [] as Set

pipeline {

    agent {
        label 'master'                                                                                                                                        
    }

    // triggers {
    //    ciBuildTrigger(
    //        noSquash: true,
    //        providerList: [
    //            rabbitMQSubscriber(
    //                name: env.FEDORA_CI_MESSAGE_PROVIDER,
    //                overrides: [
    //                    topic: 'org.fedoraproject.prod.bodhi.update.status.testing.koji-build-group.build.complete',
    //                    queue: 'osci-pipelines-queue-11'
    //                ],
    //                checks: [
    //                    [field: '$.artifact.release', expectedValue: 'f34|f33|f32|f31']
    //                ]
    //            )
    //        ]
    //    )
    // }

    parameters {
        string(name: 'CI_MESSAGE', defaultValue: '{}', description: 'CI Message')
    }

    stages {
        stage('Trigger Testing') {
            steps {
                script {
                    msg = readJSON text: params.CI_MESSAGE

                    if (msg) {
                        def releaseId = msg['artifact']['release']
                        if (releaseId == env.FEDORA_CI_RAWHIDE_RELEASE_ID) {
                            // this is rawhide
                            releaseId = 'master'
                        }

                        msg['artifact']['builds'].each { build ->
                            allTaskIds.add(build['task_id'])
                        }

                        if (allTaskIds) {
                            allTaskIds.each { taskId ->
                                artifactId = "koji-build:${taskId}"
                                additionalArtifactIds = allTaskIds.findAll{ it != taskId }.collect{ "koji-build:${it}" }.join(',')

                                build job: "fedora-ci/dist-git-pipeline/${releaseId}", wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId), string(name: 'ADDITIONAL_ARTIFACT_IDS', value: additionalArtifactIds) ]
                            }
                        }
                    }
                }
            }
        }
    }
}
