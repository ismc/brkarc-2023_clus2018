properties([[$class: 'HudsonNotificationProperty',
    endpoints: [[buildNotes: '', urlInfo: [urlOrId: ' http://cisco-spark-integration-management-ext.cloudhub.io/api/hooks/8fde6043-69b6-11e8-bf37-06c25f4e7996', urlType: 'PUBLIC']]]
]])
pipeline {
    agent any
    options {
      disableConcurrentBuilds()
      skipDefaultCheckout()
      ansiColor('xterm')
      lock resource: 'jenkins-wan-testbed'
    }
    environment {
      ANSIBLE_INVENTORY_DIR = "${env.WORKSPACE}/inventory"
      GIT_COMMITTER_NAME = 'scarter-jenkins'
      GIT_COMMITTER_EMAIL = 'jenkins@ismc.io'
    }
    triggers {
      cron('H 2 * * 1-5')
    }
    stages {
        stage('Checkout Code') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: scm.branches,
                    doGenerateSubmoduleConfigurations: false,
                    extensions: scm.extensions + [[$class: 'SubmoduleOption',
                        parentCredentials: true,
                        disableSubmodules: false,
                        recursiveSubmodules: true,
                        reference: '',
                        trackingSubmodules: false]],
                    /* userRemoteConfigs: scm.userRemoteConfigs */
                    userRemoteConfigs: [[credentialsId: 'scarter-jenkins_key', url: 'git@github.com:ismc/brkarc-2023_clus2018.git']]
                ])
                checkout([$class: 'GitSCM',
                    branches: [[name: '*/master']],
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'inventory']],
                    submoduleCfg: [],
                    userRemoteConfigs: [[credentialsId: 'scarter-jenkins_key', url: 'git@github.com:ismc/inventory-test.git']]
                ])
            }
        }
/*
        stage('Destroy Testbed') {
            steps {
                script {
                    try {
                        echo 'Destroying Cloud...'
                        retry(3) {
                            withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
                                ansiblePlaybook colorized: true, extras: "-e cloud_model=wan-testbed -e cloud_inventory_dir=${env.ANSIBLE_INVENTORY_DIR} -e cloud_project=scarter -e cloud_key_name=scarter-jenkins", playbook: 'destroy-cloud.yml'
                            }
                        }
                    } catch (e) {
                        echo 'Previous testbed not found!'
                    }
                }
            }
        }
*/
        stage('Build Testbed') {
            steps {
                echo 'Building Cloud...'
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'Ansible (scarter)']]) {
                  ansiblePlaybook colorized: true, extras: "-e cloud_model=wan-testbed -e cloud_inventory_dir=${env.ANSIBLE_INVENTORY_DIR} -e cloud_project=scarter -e cloud_key_name=scarter-jenkins", playbook: 'build-cloud.yml'
                }
                script {
                    try {
                        echo 'Updating Inventory...'
                        dir ('inventory') {
                            sshagent (credentials: ['scarter-jenkins_key']) {
                                sh 'git add *'
                                sh 'git commit -am "Updated inventory on re-build"'
                                sh 'git push origin HEAD:master --force'
                            }
                        }
                    } catch (e) {
                            echo 'Unable to commit inventory'
                    }
                }
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Running network-system.yml...'
                ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-testbed.yml", playbook: 'network-system.yml'

                echo 'Running network-checkpoint.yml...'
                ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-testbed.yml", playbook: 'network-checkpoint.yml'

                echo 'Running network-dmvpn.yml...'
                ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-testbed.yml", playbook: 'network-dmvpn.yml'

                echo 'Running network-dmvpn-check.yml...'
                ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-testbed.yml", playbook: 'network-dmvpn-check.yml'

                echo 'Running network-rollback.yml...'
                ansiblePlaybook credentialsId: 'scarter-jenkins_key', colorized: true, disableHostKeyChecking: true, inventory: "${env.ANSIBLE_INVENTORY_DIR}/wan-testbed.yml", playbook: 'network-rollback.yml'
            }
        }
        stage('Clean Workspace') {
                steps {
                echo 'Cleaning Workspace...'
                deleteDir()
            }
        }
    }
}
