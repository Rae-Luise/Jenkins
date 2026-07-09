@library('shared-library')_


pipeline {
    agent any
    tools {
        nodejs "node"
    }
    stages {
        stage('increment version') {
            steps {
                script {
                    // enter app directory, because that's where package.json is located
                    dir("app") {
                        increment_version()
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    //enter app directory, because that's where package.json and tests are located
                    dir("app") {
                        run_tests()
                    }
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                build_and_push_docker_image()
                 }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    commit_version_update()

                    }
                }
            }
        }
    }
}
