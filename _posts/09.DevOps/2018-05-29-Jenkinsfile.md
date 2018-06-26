---
title: "Jenkisfile"
categories: DevOps
tags: [DevOps, Jenkisfile]
published: false
comments: true
---



# Declarative Pipeline

기존에 Jenkinsfile은 아래와 같이 정의를 하였습니다.

```
node {
  ....
}
```

이 방식은 Pipeline Plugin 2.5버전 이전에서 사용하던 Scripted Pipeline 기법입니다. 최근에는 Declarative Pipeline 기법이 도입되었고 다음과 같습니다.

```
pipeline {
  ....  
}
```





```
pipeline {
    agent any
    // 상수, 변수 정의
    environment {
        APPLICATION_NAME     = 'heartbeat'
        DOCKER_REGISTRY_HOST = 'dkr.mrblue.com'
        DEPLOY_SERVER_HOST   = '211.214.161.24'
    }

    // stages 블록 중 하나 이상의 stage를 정의
    stages {
        stage('Compile') {
            steps {
                gradlew("clean", "classes")
            }
        }
        stage('Unit test') {
            steps {
                gradlew("testClasses")
            }
        }
        stage('Static analysis') {
            steps {
                sh "echo Static analysis stage starting"
            }
        }
        stage('build') {
            steps {
                gradlew("build")
            }
        }
        stage('Create docker image') {
            // when 블록에서 stage를 실행하는 조건을 지정할 수다다.
            when  {
                // 빌드 및 테스트 실패시 배포하지 않음.
                expression {
                    currentBuild.currentResult == 'SUCCESS'
                }
            }
            steps {
                sh("docker build -t ${DOCKER_REGISTRY_HOST}/${APPLICATION_NAME}:latest .")
            }
        }
        stage('Push docker image') {
            when {
                expression {
                    currentBuild.currentResult == 'SUCCESS'
                }
            }
            steps {
                sh("docker push ${DOCKER_REGISTRY_HOST}/${APPLICATION_NAME}:latest")
            }
        }
        stage('Deploy') {
            parallel {
                stage('Deploy to devel') {
                    when {
                        branch 'develop'
                    }
                    steps {
                        echo 'Deploying to devel'
                    }
                }
                stage('Deploy to staging') {
                    when {
                        branch 'staging'
                    }
                    steps {
                        echo 'Deploying to staging'
                    }
                }
                stage('Deploy to production') {
                    when {
                        branch 'master'
                    }
                    steps {
                        echo 'Deploying to production'
                        sshagent(['211.214.161.24-cicd']) {
                            sh "ssh cicd@${DEPLOY_SERVER_HOST} 'echo WpszlstmqovhC!C@ | sudo -S nohup bash /usr/local/service/mrblue_heartbeat/deploy.sh'"
                        }
                    }
                }
            }
        }
    }

    // 모든 steps 프로세스 이후 작업을 정의
    post {
        always {
            // 작업공간 디렉토리 삭제
            deleteDir()

            // 도커 이미지 삭제
            sh("docker rmi ${DOCKER_REGISTRY_HOST}/${APPLICATION_NAME}:latest")
        }
        success {
            slackSend channel: '#jenkins',
                      color: 'good',
                      message: "The pipeline ${currentBuild.fullDisplayName} completed successfully.\n${env.BUILD_URL}"
        }
        failure {
            slackSend channel: '#jenkins',
                      color: 'danger',
                      message: "The pipeline ${currentBuild.fullDisplayName} failed.\n${env.BUILD_URL}"
        }
    }
}

def gradlew(String... args) {
    sh "./gradlew ${args.join(' ')}"
}
```



