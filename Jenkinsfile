pipeline{
    agent any
    environment{
        VERSION = "$env.BUILD_ID"
    }
    stages{
        stage("sonarqube quality check"){
            agent{
                docker {
                    image 'openjdk:11'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
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
        stage("docker build && docker push"){
            steps{
                withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_pass')]) {
                    sh '''
                    docker build -t 192.168.198.137:8083/springapp:${VERSION} .
                    docker login -u admin -p ${docker_pass} 192.168.198.137:8083
                    docker push 192.168.198.137:8083/springapp:${VERSION}
                    docker rmi 192.168.198.137:8083/springapp:${VERSION}
                    '''
            }
                
            }
        }
    }
}