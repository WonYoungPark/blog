---
title: "Jenkins Guide Tour"
categories: DevOps
tags: [DevOps, Jenkins, Jenkins Pipeline]
published: true
comments: true
---



> 해당 글은 [jenkins guide tour](https://jenkins.io/doc/pipeline/tour/getting-started/) 를 요약 및 정리한 글입니다.



## What is a Jenkins Pipeline?

Jenkins에 지속적인 전송 파이프 라인을 구현하고 통합하는 것을 지원하는 플러그인 모음입니다. 젠킨스 파일을 통해서 우리는 자동배포 프로세스를 git과 같은 형상관리툴로 관리할 수 있습니다.

젠킨스 파일은 일반적으로 텍스트로 작성되어 지며, 단순하거나 복잡한 파이프 라인 프로세스를 **코드** 로 정의할 수 있도록 제공해 줍니다.

우리는 프로젝트에 Jenkinsfile을 생성하여 파이프라인을 정의 하였다고 한다면, 젠킨스는 Webhook이나 pull reqeust를 통해 새로운 이벤트를 감지하고 파이프 라인을 실행합니다.



## Running multiple steps

파이프 라인은 여러 단계로 구성할 수 있어서 애플리케이션을 테스트 및 배포 할 수 있습니다. Jenkins Pipeline을 사용하면 여러 단계를 손쉽게 구성할 수 있으므로 어떠한 종류의 자동화 프로세스라고 하더라도 모델링 할 수 있습니다.

`step` 을 단일 조치를 수행하는 단일 명령과 같다고 생각해 보십시오. `step` 이 성공하면 다음 `step` 으로 넘어가고, `step` 정상적으로 실행되지 않으면 파이프라인은 실패한 것으로 간주합니다.



### Linux, BSD, and Mac OS

리눅스 OS 시스템에서 `sh` 명령어 실행을 파이프라인을 통해 다음과 같이 실행시킬 수 있습니다.

```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'echo "Hello World"'
                sh '''
                    echo "Multiline shell steps works too"
                    ls -lah
                '''
            }
        }
    }
}
```



### Timeouts, retries and more

성공할 때까지 단계를 다시 시도하거나 너무 올래 걸리는 단계와 같은 문제를 쉽게 해결할 수 있는 강력한 `wrap step` 을 제공합니다.

```
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                retry(3) {
                    sh './flakey-deploy.sh'
                }

                timeout(time: 3, unit: 'MINUTES') {
                    sh './health-check.sh'
                }
            }
        }
    }
}
```



### Finishing up

파이프라인 실행이 끝났을 때 파이파인 프로세스 수행 결과에 따라 특정 이벤트 처리를 해야할 경우가 있습니다. `post section` 을 통해 우리는 이러한 작업을 수행할 수 있습니다.

```
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'echo "Fail!"; exit 1'
            }
        }
    }
    post {
        always {
            echo 'This will always run'
        }
        success {
            echo 'This will run only if successful'
        }
        failure {
            echo 'This will run only if failed'
        }
        unstable {
            echo 'This will run only if the run was marked as unstable'
        }
        changed {
            echo 'This will run only if the state of the Pipeline has changed'
            echo 'For example, if the Pipeline was previously failing but is now successful'
        }
    }
}
```



## Using environment variables

변수는 아래의 예제처럼 전역적으로 선언할 수 있습니다. `stage`  별로 변수를 선언하고 싶을 경우 `stage` 내부에 변수를 정의하면 됩니다.

```
pipeline {
    agent any

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Build') {
            steps {
                sh 'printenv'
            }
        }
    }
}
```



## Cleaning up and notifications

위에 언급했던 `post` 의 사용방법에 대해서 나누어 보도록 하겠습니다. `post` 섹션을 파이프 라인의 실행 결과에 따라서 다른데 동작하기 때문에 이를 이용하여, 알림 기능을 사용할 수 있습니다.

```
pipeline {
    agent any
    stages {
        stage('No-op') {
            steps {
                sh 'ls'
            }
        }
    }
    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'I succeeeded!'
        }
        unstable {
            echo 'I am unstable :/'
        }
        failure {
            echo 'I failed :('
        }
        changed {
            echo 'Things were different before...'
        }
    }
}
```



### Email

```
post {
    failure {
        mail to: 'team@example.com',
             subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
             body: "Something is wrong with ${env.BUILD_URL}"
    }
}
```



### Slack

```
post {
    success {
        slackSend channel: '#ops-room',
                  color: 'good',
                  message: "The pipeline ${currentBuild.fullDisplayName} completed successfully."
    }
}
```