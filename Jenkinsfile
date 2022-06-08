pipeline {
    
    agent {
        label "Jenkins-slave"
    }
    
    
    stages {
        stage('SCM') {
            steps {
                git 'https://github.com/pramodvpai/jenkins-docker-maven-java-webapp.git'
                
            }
            
        }
        
        stage('Build by Maven Package') {
            steps {
                sh 'mvn clean package'
            }
            
        }
        
        
        stage('Build Docker OWN image') {
            steps {
                sh "sudo docker build -t  pramodvpp/javaweb:${BUILD_TAG}  ."
                //sh 'whoami'
            }
            
        }
        
        
        stage('Push Image to Docker HUB') {
            steps {
                
                withCredentials([string(credentialsId: 'DOCKER_HUB_PWD', variable: 'DOCKER_HUB_PASS_CODE')]) {
    // some block
                 sh "sudo docker login -u pramodvpp -p $DOCKER_HUB_PASS_CODE"
}
               
               sh "sudo docker push pramodvpp/javaweb:${BUILD_TAG}"
            }
            
        }
        
        
        stage('Deploy webAPP in DEV Env') {
            steps {
                sh 'sudo docker rm -f myjavaapp'
                ansiblePlaybook credentialsId: 'ansible-master-slave', disableHostKeyChecking: true,extras: "-e BUILD_TAG=${BUILD_TAG}", installation: 'Ansible', inventory: 'dev.inv', playbook: 'docker_deploy'
                //sh 'whoami'
            }
            
        }
        
        
        stage('Deploy webAPP in QA/Test Env') {
            steps {
               
               sshagent(['QA_ENV_SSH_CRED']) {
    
                    sh "ssh  -o  StrictHostKeyChecking=no ec2-user@13.126.242.227 sudo docker rm -f myjavaapp"
                    sh "ssh ec2-user@13.126.242.227 sudo docker run  -d  -p  8080:8080 --name myjavaapp   pramodvpp/javaweb:${BUILD_TAG}"
                }

            }
            
        }
        
        
         stage('QAT Test') {
            steps {
            retry(5) {
                sh 'curl --silent http://13.126.242.227:8080/java-web-app/ |  grep India'
                sh 'echo >/dev/tcp/13.126.242.227/8080 && echo "port $port is open"'
                sh 'curl -ls http://13.126.242.227:8080 | head -1'
                sh 'curl -s -o /dev/null -w "%{http_code}" http://13.126.242.227:8080/java-web-app/'
                       }
    }
        
            }
         
        
        stage('approved') {
            steps {
                
            
            script {
                Boolean userInput = input(id: 'Proceed1', message: 'Promote build?', parameters: [[$class: 'BooleanParameterDefinition', defaultValue: true, description: '', name: 'Please confirm you agree with this']])
                echo 'userInput: ' + userInput

                if(userInput == true) {
                    // do action
                } else {
                    // not do action
                    echo "Action was aborted."
                }
            
                
            }
        }
        }
        
        stage('Deploy webAPP in Prod Env') {
            steps {
               
               sshagent(['QA_ENV_SSH_CRED']) {
    
                    sh "ssh  -o  StrictHostKeyChecking=no ec2-user@13.232.95.15 sudo docker rm -f myjavaapp"
                    sh "ssh ec2-user@13.232.95.15 sudo docker run  -d  -p  8080:8080 --name myjavaapp   pramodvpp/javaweb:${BUILD_TAG}"
                }


            }
        }
         
         stage('Cleanup') {
           steps {
             sh 'sudo docker rm -f myjavaapp'
             ansiblePlaybook credentialsId: 'ansible-master-slave', disableHostKeyChecking: true, installation: 'Ansible', inventory: 'dev.inv', playbook: 'cleanup'
                //sh 'whoami'
            }
            
        }
                
        stage('Health checker') {
            steps {
                sh 'curl http://13.126.242.227:8080/java-web-app/'
            }
        }
            stage ('Email Notification') {
                    steps {
                    emailext attachLog: true, from: 'admin@jenkins-master', body: '''Hi System Admin,Please find the status of Build of Production Env Thank you''', compressLog: true, subject: 'Post Build Status-Production', to: 'vpramodpai@yahoo.co.in'
                }
        }    }
    }
 
   
