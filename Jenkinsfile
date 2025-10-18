pipeline {
    agent {
        label 'INFRA'
    }

    environment {
        SERVER_ID = 'jfrog_java'
    }

    triggers {
        pollSCM('* * * * *')
    }

    stages {
        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/gandru123/spring-petclinic.git', branch: 'main'
            }
        }

        stage('Build Java Project') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withCredentials([string(credentialsId: 'sonar_id', variable: 'SONAR_TOKEN')]) {
                    withSonarQubeEnv('MYSONARQUBE') {
                        sh '''
                            mvn sonar:sonar \
                                -Dsonar.projectKey=gandru123_spring-petclinic \
                                -Dsonar.organization=jenkins-java \
                                -Dsonar.host.url=https://sonarcloud.io \
                                -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Upload to JFrog Artifactory') {
            steps {
                script {
                    sh 'ls -l target/*.jar || echo "NO JAR found in target"'
                    def server = Artifactory.server(SERVER_ID)
                    def buildInfo = Artifactory.newBuildInfo()

                    server.upload(spec: '''{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "newrepo-libs-release-local/"
                            }
                        ]
                    }''', 
                    buildInfo: buildInfo)

                    server.publishBuildInfo(buildInfo)
                }
            }
        }
    }

    post {
        always {
            junit '**/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: '**/target/*.jar'
        }
        success {
            echo 'Build completed successfully!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
