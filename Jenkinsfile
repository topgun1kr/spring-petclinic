pipeline {
    agent any
    
    tools {
        maven "M3"
        jdk "JDK17"
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
        stage('Maven Build') {
            steps {
                echo 'Maven Build'
                sh 'mvn -Dmaven.test.failuer.ignore=true clean package'
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
                    nohup java -jar /home/ubuntu/deploy/spring-petclinic-2.7.3.BUILD-SNAPSHOT.jar >> nohup.out 2>&1 &''',
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
