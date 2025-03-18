pipeline {
    agent any
    
    tools {
        maven "M3"
        jdk "JDK17"
    }
    environmet {
        DOCKERHUB_CREDENTIALS = credentials('dockerCredential')
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
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn -Dmaven.test.failuer.ignore=true clean package'
            }
            post {
               success {
                   junit 'target/surefire-reports/**/*.xml'
               }
            }
            
        }
        //docker image 생성
        stage('Docker Image Build') {
            steps {
                dir("${env.WORKSPACE}") {
                    sh '''
                      docker build -t spring-petclinic:$build_NUMBER .
                      docker tag spring-petclinic:$build_NUMBER topgun1kr/spring-petclinic:latest
                      '''
                }
            }
        }  
        //docker image push
        stage('Docker Image Push'){
            steps {
                sh '''
                echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $ $DOCKERHUB_CREDENTIALS_USR --password-stdin
                docker push topgun1kr/spring-petclinic:latest
                '''
            }
        }
        stage('SSH Publish') {
           steps {
                    echo 'SSH Publish'
                    sshPublisher(publishers: [sshPublisherDesc(configName: 'target',
                    transfers: [sshTransfer(cleanRemote: false,
                    excludes: '',
                    execCommand: '''
                    fuser -k 8080/tcp
                    export BUILD_ID=Petclinic-Pipeline
                    nohup java -jar spring-petclinic-3.4.0-SNAPSHOT.jar >> nohup.out 2>&1 &''',
                    execTimeout: 120000,
                    flatten: false,
                    makeEmptyDirs: false,
                    noDefaultExcludes: false,
                    patternSeparator: '[, ]+',
                    remoteDirectory: '',
                    remoteDirectorySDF: false,
                    removePrefix: 'target',
                    sourceFiles: 'target/*.jar')],
                    usePromotionTimestamp: false,
                    useWorkspaceInPromotion: false, verbose: false)])
            }
            
        }
        
    }
}
