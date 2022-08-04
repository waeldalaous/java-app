pipeline{
    agent any
    environment{
        VERSION = "$env.BUILD_ID"
    }
    stages{
        stage("sonarqube quality check"){
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
        /*stage('identifying misconfigs using datree in helm charts'){
            steps{
                script{
                    dir('kubernetes/') {
                        withCredentials([string(credentialsId: 'datree_token', variable: 'datree_token')]) {
                            sh 'helm datree config set token ${datree_token}'
                            sh 'helm datree test myapp/'

                        }
                        
                    }
                }
            }
        }*/
        stage("push helm charts to nexus"){
            steps{
                withCredentials([string(credentialsId: 'docker_pass', variable: 'docker_pass')]) {
                    dir('kubernetes/') {
                        sh '''
                        helmversion=$(helm show chart myapp/ | grep version | cut -d: -f 2 | tr -d ' ')
                        tar -czvf myapp-${helmversion}.tgz myapp/
                        curl -u admin:$docker_pass http://192.168.198.137:8081/repository/helm-repo/ --upload-file myapp-${helmversion}.tgz -v
                        '''
                        
                    }
                }
                
            }
        
        }
        stage('connecting to k8s cluster'){
	        steps{
	            script{
	    		    withCredentials([kubeconfigContent(credentialsId: 'kubernetes_config', variable: 'KUBECONFIG_CONTENT')]){
			            dir ("kubernetes/"){  
				            sh 'helm list'
				            sh 'helm upgrade --install --set image.repository="nexus_ip:8083/springapp" --set image.tag="${VERSION}" myjavaapp myapp/ ' 
			            }
	    		    }
	            }
	        }
			            
		}
    }
}