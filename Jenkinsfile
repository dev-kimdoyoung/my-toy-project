// 파이프라인 정의
pipeline {
    // 스테이지 별로 다른 거
    agent any

    triggers {
        pollSCM('*/3 * * * *')
    }

    // 젠킨스에서 사용할 환경변수 등록
    environment {
      // 젠킨스에 등록된 Credential ID에 해당하는 Value를 가져옴 (캡슐화)
      AWS_ACCESS_KEY_ID = credentials('awsAccessKeyId')
      AWS_SECRET_ACCESS_KEY = credentials('awsSecretAccessKey')
      AWS_DEFAULT_REGION = 'ap-northeast-2'
      HOME = '.' // Avoid npm root owned
    }

    stages {
        // 레포지토리를 다운로드 받음
        stage('Prepare') {
            // 에이전트는 아무거나 사용
            agent any
            
            steps {
                echo "Lets start Long Journey! ENV: ${ENV}"
                echo 'Clonning Repository'

                // Git 정보 등록
                git url: 'https://github.com/dev-kimdoyoung/my-toy-project.git',
                    branch: 'master',
                    credentialsId: 'jenkinsgit'
            }

            // Step이 끝난 이후 진행되는 작업 (사후 작업)
            post {
                // If Maven was able to run the tests, even if some of the test
                // failed, record the test results and archive the jar file.
                success {
                    echo 'Successfully Cloned Repository'
                }
            }
        }
        
        // aws s3 에 파일을 올림
        stage('Deploy Frontend') {
          steps {
            echo 'Deploying Frontend'
            // 프론트엔드 디렉토리 (/website) 의 정적 (static) 파일들을 S3 에 올림, 이 전에 반드시 EC2 instance profile 을 등록해야함.
            dir ('./website'){  // Front-End 디렉토리 지정

                // AWS S3의 dev-kim 버킷에 파일 업로드
                sh '''
                aws s3 sync ./ s3://dev-kim
                '''
            }
          }

          post {
              // If Maven was able to run the tests, even if some of the test
              // failed, record the test results and archive the jar file.

              // 빌드 성공 시 메일 발송 (젠킨스에서 SMTP 서버 정보 등록을 해주어야 함)
              success {
                  echo 'Successfully Cloned Repository'

                  mail  to: 'octl4573@gmail.com',
                        subject: "Deploy Frontend Success",
                        body: "Successfully deployed frontend!"
                  
              }

              // 빌드 실패 시
              failure {
                  echo 'I failed :('

                  mail  to: 'octl4573@gmail.com',
                        subject: "Failed Pipelinee",
                        body: "Something is wrong with deploy frontend"
              }
          }
        }
        
        stage('Lint Backend') {
            // Docker plugin and Docker Pipeline 두개를 깔아야 사용가능!
            agent {
              // Slave 노드 : DockerHub에서 node 이미지를 가져와서 해당 stage 환경 구성
              docker {
                image 'node:latest'
              }
            }
            
            steps {
              // 코드 lint 확인
              dir ('./server'){
                  sh '''
                  npm install&&
                  npm run lint
                  '''
              }
            }
        }
        
        // 테스트 코드 돌리기
        stage('Test Backend') {
          agent {
            docker {
              image 'node:latest'
            }
          }
          steps {
            echo 'Test Backend'

            // 테스트 코드 run
            dir ('./server'){
                sh '''
                npm install
                npm run test
                '''
            }
          }
        }
        
        // 서버 코드를 도커로 빌드
        stage('Bulid Backend') {
          agent any
          steps {
            echo 'Build Backend'

            dir ('./server'){
                // PROD : Product 환경에서 배포
                sh """
                docker build . -t server --build-arg env=${PROD}
                """
            }
          }

          post {
            failure {
              error 'This pipeline stops here...'
            }
          }
        }
        
        stage('Deploy Backend') {
          agent any

          steps {
            echo 'Build Backend'

            dir ('./server'){
                sh '''
                docker rm -f $(docker ps -aq)
                docker run -p 80:80 -d server
                '''
            }
          }

          post {
            success {
              mail  to: 'frontalnh@gmail.com',
                    subject: "Deploy Success",
                    body: "Successfully deployed!"
                  
            }
          }
        }
    }
}
