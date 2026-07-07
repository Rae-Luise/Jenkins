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
                        // update application version in the package.json file with one of these release types: patch, minor or major
                        // This command updates the minor version in package.json and ensures no Git commands are executed in the background, preventing automatic commits or tags in your Jenkins Pipeline
                        sh "npm version minor --no-git-tag-version"

                        // read the updated version from the package.json file
                        def packageJson = readJSON file: 'package.json'
                        def version = packageJson.version

                        // set the new version as part of IMAGE_NAME
                        env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                    }

                    // alternative solution without Pipeline Utility Steps plugin:
                    // def version = sh (returnStdout: true, script: "grep 'version' package.json | cut -d '\"' -f4 | tr '\\n' '\\0'")
                    // env.IMAGE_NAME = "$version-$BUILD_NUMBER"
                }
            }
        }
        stage('Run tests') {
            steps {
               script {
                    //enter app directory, because that's where package.json and tests are located
                    dir("app") {
                        //install all dependencies needed for running tests
                        sh "npm install"
                        sh "npm run test"
                    }
               }
            }
        }
        stage('Build and Push docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]){
                    sh "docker build -t luiserae/docker-project:${IMAGE_NAME} ."
                    sh 'echo $PASS | docker login -u $USER --password-stdin'
                    sh "docker push luiserae/docker-project:${IMAGE_NAME}"
                }
            }
        }
        stage('commit version update') {
            steps {
                script {
                    //withCredentials([usernamePassword(credentialsId: 'gitlab-credentials', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        //git config here for the first time run
                        sh 'git config --global user.email "jenkins@example.com"'
                        sh 'git config --global user.name "jenkins"'
                        sh 'git add .'
                        //新增这一行：强制切换到 main 分支
                        //sh 'git checkout main'
                        //sh "git remote set-url origin https://${USER}:${PASS}@github.com/Rae-Luise/Jenkins.git"
                        //sh 'git add .'
                        sh 'git commit -m "auto update by jenkins"'
                        sh "git remote set-url origin git@github.com:Rae-Luise/Jenkins.git"
                        sh 'mkdir -p ~/.ssh && ssh-keyscan github.com >> ~/.ssh/known_hosts'
                        sh 'git push -f origin HEAD:main'

                    }
                }
            }
        }
    }

