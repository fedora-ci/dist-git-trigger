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
                    msg = readJSON text: CI_MESSAGE

                    if (msg) {
                        def srpm = msg['srpm']
                        def prIdList = srpm.split('#')[0].split(';')
                        def prId = "fedora-dist-git:${prIdList[1]}@${prIdList[2]}#${prIdList[3]}"
                         // TODO: how reliable is this? the second item in the array *should* be the release Id
                        def branch = msg['info']['request'][1]

                        if (branch == env.FEDORA_CI_RAWHIDE_RELEASE_ID) {
                            // FIXME: once we move replace master branch with fmf
                            branch = 'fmf'
                        }

                        artifactId = "(koji-build:${msg['id']})->${prId}"
                        build job: 'fedora-ci/dist-git-pipeline/${branch}', wait: false, parameters: [ string(name: 'ARTIFACT_ID', value: artifactId) ]
                    }
                }
            }
        }
    }
}
