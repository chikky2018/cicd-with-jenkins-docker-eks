pipeline {
    agent any

    stages {
        stage('Clone git repo') {
            steps {
                sh 'echo "STAGE 0: Cloning app code from SCM ..."'
                git 'https://github.com/mudathirlawal/cicd-with-jenkins-docker-and-aws-eks.git'
            }    
        }        
        stage('Lint all app code') {
            steps {
                sh 'echo "STAGE 1: Checking app code for syntax error ..."'
                sh 'tidy -q -e *.html'
            }
        }   
        stage( 'Build docker image for app' ) {
            steps {
                sh 'echo "STAGE 2: Building and tagging docker image ..."'
                sh 'docker build -t web-app'
                sh 'docker image ls'                  
            }
        } 
        stage( 'Push image to dockerhub repo' ) {
            steps {
                withDockerRegistry([url: "", credentialsId: "dockerhub"]) {
                    sh 'echo "STAGE 3: Uploading image to dockerhub repository ..."'
                    sh 'docker login'
                    sh 'docker tag web-app:v1.0 chikky2018/web-app'
                    sh 'docker push chikky2018/web-app:v1.0'          
                }
            }
        }                                   
        stage( 'Deploy image to AWS EKS' ) {
            steps {
                withAWS( region:'us-east-1', credentials:'k8s' ) {
                    sh 'echo "STAGE 4: Deploying image to AWS EKS cluster ..."'
                    sh 'aws eks --region us-east-1 update-kubeconfig --name k8s'
                    sh 'kubectl config use-context arn:aws:eks:us-east-1:878972269374:cluster/dev'
                    sh 'kubectl set image deployment web-app web-app=chikky2018/web-app'
                    sh 'kubectl rollout status deployment web-app'
                    sh 'kubectl apply -f templates/deployment.yml'
                    sh 'kubectl apply -f templates/loadbalancer.yml'
                    sh 'kubectl apply -f templates/aws-auth-cm.yml'
                    sh 'kubectl get nodes --all-namespaces'
                    sh 'kubectl get deployment'
                    sh 'kubectl get pod -o wide'
                    sh 'kubectl get service/web-app'
                    sh 'echo "Congratulations! Deployment successful."'
                    sh 'kubectl describe deployment/web-app'
                }
            }
        }               
    }
}
