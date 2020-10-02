#!groovy


def msg
def artifactId

pipeline {

    agent {
        label 'master'
    }

    triggers {
        ciBuildTrigger(
            noSquash: true,
            providerList: [
                rabbitMQSubscriber(
                    name: env.FEDORA_CI_MESSAGE_PROVIDER,
                    overrides: [
                        topic: 'org.fedoraproject.prod.buildsys.task.state.change',
                        queue: 'osci-pipelines-queue-11'
                    ],
                    checks: [
                        [field: '$.owner', expectedValue: 'bpeck/jenkins-continuous-infra.apps.ci.centos.org'],
                        [field: '$.new', expectedValue: 'CLOSED'],
                        [field: '$.method', expectedValue: 'build'],
                        [field: '$.srpm', expectedValue: '^fedora-ci_*']
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
                    msg = readJSON text: CI_MESSAGE

                    if (msg) {
                        def srpm = msg['srpm']
                        def prIdList = srpm.split('#')[0].split('_')
                        def prId = "fedora-dist-git:${prIdList[1]}@${prIdList[2]}#${prIdList[3]}"

                        artifactId = "(koji-build:${msg['id']})->${prId}"
                        build job: 'fedora-ci/dist-git-pipeline/tmt', wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId) ]
                    }
                }
            }
        }
    }
}
