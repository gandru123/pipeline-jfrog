pipeline{
    agent{
        label 'INFRA'
    }
    triggers {
        pollSCM ('* * * * *')
    }
    stages{
        stage('git repo'){
            steps {
                git url: 'https://github.com/gandru123/spring-petclinic.git',branch:'main'
            }
        }
        stage('install pulgin') {
            steps{
                sh 'mvn clean package'
            }
        }
        stage('configure sonarqube') {
            steps{
                withCredentials([string(credentialsId:'sonar_id', variable:'SONAR_TOKEN')]){
                    withSonarQubeEnv('MYSONARQUBE'){
                        sh '''
                            mvn package sonar:sonar \
                            -Dsonar.projectKey=gandru123_spring-petclinic \
                            -Dsonar.organization=jenkins-java \
                            -Dsonar.host.url=https://sonarcloud.io \
                            -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
            
            }
        }
        stage('configure jfrog'){
            steps{
                script {
                    def server = Artifactory.server 'jfrog_id'
                    def buildInfo = Artifactory.newBuildInfo()

                    server.upload(
                        spec: '''{
                            "files": [
                                {
                                    "parttern": "target/*.jar" ,
                                    "target" : "newrepo-libs-release-local/"
                                    }
                                ]
                            }''',
                            buildInfo: buildInfo
                    )
                    server.publishBuildInfo(buildInfo)
                    

                }
            }
        }
    }
    post{
        always{
            junit '**/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: '**/target/*.jar'
        }
        success{
            echo 'build is success'
        }
        failure{
            echo 'build is fail'
        }
    }

}