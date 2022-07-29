pipeline{
    agent any
    stages{
        stage("sonarqube quality check"){
            agent{
                docker {
                    image 'openjdk'
                }
            }
            steps{
                script{
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh 'chmod +x gradlew'
                        sh './gradlew sonarqube'
                    }

                }
            }
            
        }
    }
}