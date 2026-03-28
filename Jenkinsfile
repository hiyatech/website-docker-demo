pipeline {  
    agent any  
  
    environment {  
        AWS_REGION = 'ap-south-1'  
        ECR_REPO = 'website-docker-demo'  
        AWS_ACCOUNT_ID = '117750149494'  
        IMAGE_TAG = "${env.BUILD_NUMBER}"  
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"  
        LATEST_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest"  
        DEPLOY_SERVER = '3.111.217.79'  
    }  
  
    stages {  
        stage('Checkout') {  
            steps {  
                git branch: 'main', url: 'https://github.com/hiyatech/website-docker-demo.git'  
            }  
        }  
  
        stage('Build Docker Image') {  
            steps {  
                sh 'docker build -t website-docker-demo .'  
            }  
        }  
  
        stage('Tag Docker Image') {  
            steps {  
                sh '''  
                    docker tag website-docker-demo:latest $IMAGE_URI  
                    docker tag website-docker-demo:latest $LATEST_URI  
                '''  
            }  
        }  
  
        stage('Login to ECR') {  
            steps {  
                sh '''  
                    aws ecr get-login-password --region $AWS_REGION |  
                    docker login --username AWS --password-stdin  
                    $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com  
                '''  
            }  
        }  
  
        stage('Push Image to ECR') {  
            steps {  
                sh '''  
                    docker push 117750149494.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:4
docker push 117750149494.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:latest 
                '''  
            }  
        }  
  
        stage('Deploy to EC2') {  
            steps {  
                sshagent(['deploy-ec2-key']) {  
                    sh '''  
                        ssh -o StrictHostKeyChecking=no ubntu@$DEPLOY_SERVER "  
                        aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 117750149494.dkr.ecr.ap-south-1.amazonaws.com
                        docker pull $LATEST_URI &&  
                        docker stop website-demo || true &&  
                        docker rm website-demo || true &&  
                        docker run -d --name website-demo -p 80:80 $LATEST_URI  
                        "  
                    '''  
                }  
            }  
        }  
    }  
  
    post {  
        success {  
            echo 'Website deployed successfully'  
        }  
        failure {  
            echo 'Pipeline failed'  
        }  
    }  
}

