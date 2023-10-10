pipeline{
    agent any
    environment{
        VERSION = "${env.BUILD_ID}"
    }
    stages{
        stage("Sonar Quality Check"){
            agent {
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarserver') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }
                    timeout(time: 1, unit: 'HOURS') {
                      def qg = waitForQualityGate()
                      if (qg.status != 'OK') {
                           error "Pipeline aborted due to quality gate failure: ${qg.status}"
                      }
                    }
                }
            }
            
        }
    }
    stage( "Docker Build & Docker Push to Nexus" ){
        steps{
            script{
                withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_password')]) {
                    sh '''
                        docker build -t 34.142.233.78:8083/cicd2-docker-nexus:${VERSION} .
                        docker login -u admin -p $docker_password 34.142.233.78:8083
                        docker push 34.142.233.78:8083/cicd2-docker-nexus:${VERSION}
                        docker rmi 34.142.233.78:8083/cicd2-docker-nexus:${VERSION}
                    '''
                }
            }
        }
    }
    post{
        always{
            echo "CICD Pipeline SUCCESS"
        }
    }
}