pipeline {
    // 스테이지 별로 다른 거
    agent any

    // 3분 주기마다 github에서 clone
    // Jenkins 서버 웹페이지에서 파이프라인 생성 시 'Poll SCM' 기능을 활성화 해야만 아래 코드가 작동함
    triggers {
        pollSCM('*/3 * * * *')
    }

    environment {
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
    }

    stages {
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            agent any
            
            steps {
                echo 'Clonning Repository'

                git url: 'https://github.com/dev-kimdoyoung/my-toy-project.git',
                    branch: 'master',
                    credentialsId: 'sk-planet-jenkins'
            }

            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Successfully Cloned Repository'
                }

                always {
                  echo "i tried..."
                }

                cleanup {
                  echo "after all other post condition"
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리의 정적파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            // 실제 운영 환경 상에서는 코드 린트 검사와 같은 정적 테스트를 진행해야 함
            dir ('./website'){
                sh '''
                aws s3 sync ./ s3://kimdoyoung-jenkins-test
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.
              success {
                  echo 'Successfully Cloned Repository'

                  // stage 성공 시 사후 작업 (아래 메일 주소로 메일 전송)
                  // 메일 발송을 위해 Jenkins 서버에서 SMTP 서버 정보를 등록해야 함
                  // mail  to: 'octl4573@gmail.com',
                  //       subject: "Deploy Frontend Success",
                  //       body: "Successfully deployed frontend!"

              }

              failure {
                  echo 'I failed :('

                  // stage 실패 시 사후 작업
                  // mail  to: 'octl4573@gmail.com',
                  //       subject: "Failed Pipelinee",
                  //       body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              // node.js 공식 이미지로 도커 컨테이너 생성
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              // 자바 라이브러리 설치 및 코드 린트 검사 수행 
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
        }
        
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            // 자바 라이브러리 설치 및 테스트 수행
            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            // 서버 코드를 도커로 빌드
            dir ('./server'){
                sh """
                sudo docker build . -t server --build-arg env=${PROD}
                """
            }
          }

          post {
            // 서버 빌드 실패 시 에러 메시지 출력하고, 배포 중단
            // 해당 코드가 없을 경우, 빌드가 실패했음에도 불구하고 코드 배포를 수행하는 문제가 발생
            failure {
              error 'This pipeline stops here...'
            }
          }
        }

        // 서버 코드 배포 실행        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'

            // 실행 중인 컨테이너 종료 후 새로 배포된 코드로 컨테이너 재실행
            dir ('./server'){
                sh '''
                sudo docker rm -f $(docker ps -aq)
                sudo docker run -p 80:80 -d server
                '''
            }
          }

          // post {
          //   success {
          //     mail  to: 'octl4573@gmail.com',
          //           subject: "Deploy Success",
          //           body: "Successfully deployed!"
                  
          //   }
          // }
        }
    }
}