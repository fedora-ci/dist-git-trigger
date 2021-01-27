#!groovy

def supportedReleases = ['f32', 'f33', 'f34', 'f35', 'f36', 'f37', 'f38', 'f39']

def msg
def artifactId

pipeline {

    agent {
        label 'dist-git-trigger'
    }

    options {
        buildDiscarder(logRotator(daysToKeepStr: '45', artifactNumToKeepStr: '100'))
        skipDefaultCheckout()
    }

    triggers {
        ciBuildTrigger(
            noSquash: true,
            providerList: [
                rabbitMQSubscriber(
                    name: env.FEDORA_CI_MESSAGE_PROVIDER,
                    overrides: [
                        topic: 'org.fedoraproject.prod.buildsys.task.state.change',
                        queue: 'osci-pipelines-queue-6'
                    ],
                    checks: [
                        [field: '$.owner', expectedValue: 'bpeck/jenkins-continuous-infra.apps.ci.centos.org'],
                        [field: '$.new', expectedValue: 'CLOSED'],
                        [field: '$.method', expectedValue: 'build'],
                        [field: '$.srpm', expectedValue: '^fedora-ci_.*']
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
                    msg = readJSON text: params.CI_MESSAGE

                    if (msg) {
                        def srpm = msg['srpm']
                        def prIdList = srpm.split(';')[0].split('_')
                        def prId = "fedora-dist-git:${prIdList[1]}@${prIdList[2]}#${prIdList[3]}"
                         // TODO: how reliable is this? the second item in the array *should* be the release Id
                        def releaseId = msg['info']['request'][1]

                        if (supportedReleases.contains(releaseId)) {

                            artifactId = "(koji-build:${msg['id']})->${prId}"
                            build(
                                job: "fedora-ci/dist-git-pipeline/master",
                                wait: false,
                                parameters: [
                                    string(name: 'ARTIFACT_ID', value: artifactId),
                                    string(name: 'TEST_PROFILE', value: releaseId)
                                ]
                            )
                        }
                    }
                }
            }
        }
    }
}
