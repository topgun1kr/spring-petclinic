pipeline {
    agent any
    
    tools {
        maven "M3"
        jdk "JDK21"
    }

    environment {
        DOCKERHUB_CREDENTIALS = credentials('dockerCredential')
        REGION = "ap-northeast-2"
        AWS_CREDENTIALS_NAME = "AWSCredentials"
    }
    
    stages {
        stage('Git Clone') {
            steps {
                echo 'Git Clone'
                git url: 'https://github.com/topgun1kr/spring-petclinic.git',
                    branch: 'main'
            }
            post {
                success {
                    echo 'Git Clone Success'
                }
                failure {
                    echo 'Git Clone Fail'
                }
            }
        }
        // Maven build 작업
        stage('Maven Build') {
            steps {
                echo 'Maven Build' 
                sh 'mvn -Dmaven.test.failure.ignore=true clean package' // Test error 무시
            }            
        }

        // Docker Image 생성
        stage('Docker Image Build') {
            steps {
                echo 'Docker Image Build'
                dir("${env.WORKSPACE}") {
                   sh '''
                      docker build -t spring-petclinic:$BUILD_NUMBER .
                      docker tag spring-petclinic:$BUILD_NUMBER topgun1kr/spring-petclinic:latest
                      '''
                }
            }
        }

        // Docker Image Push
        stage('Docker Image Push') {
            steps { 
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push topgun1kr/spring-petclinic:latest
                '''
            }
        }

        // Remove Docker Image
        stage('Remove Docker Image') {
            steps {
                sh '''
                docker rmi spring-petclinic:$BUILD_NUMBER
                docker rmi topgun1kr/spring-petclinic:latest
                '''
            }
        }        
        stage('Upload S3') {
        steps {
            echo "Upload to S3"
            dir("${env.WORKSPACE}") {
                sh 'zip -r scripts.zip ./scripts appspec.yml'
                withAWS(region:"${REGION}",credentials:"${AWS_CREDENTIALS_NAME}"){
                    s3Upload(file:"scripts.zip", bucket:"team2-codedeploy-bucket")
                    }
                    sh 'rm -rf ./scripts.zip' 
                }
            }    
        }
        stage('Codedeploy Workload') {
        steps {
            echo "Codedeploy Workload"   
                withAWS(region:"${REGION}",credentials:"${AWS_CREDENTIALS_NAME}"){
                    sh '''
                    aws deploy create-deployment --application-name TEAM2_deploy \
                    --deployment-config-name CodeDeployDefault.OneAtATime \
                    --service-role-arn ############
                    --deployment-group-name TEAM2_deploy_group \
                    --ignore-application-stop-failures \
                    --s3-location bucket=team2-codedeploy-bucket,bundleType=zip,key=scripts.zip
                    '''
                }
                sleep(10) // sleep 10s
            }
        }

    }
}
    
        
         
          
