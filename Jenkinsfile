pipeline {
    environment {
        DOCKER_ID = "kentronic"
        DOCKER_IMAGE_RABBITMQ = "rabbitmq"
        DOCKER_TAG = "${BUILD_ID}"
        BUILD_AGENT  = ""
        NAMESPACE = credentials("NAMESPACE")
    }
agent any
    stages {
        stage('Build') {
            steps { //create a loop somehow??
            sh 'docker build -t $DOCKER_ID/$DOCKER_IMAGE_RABBITMQ:$DOCKER_TAG .'
                   }
        }
        stage('Run') {
            steps {
                sh 'docker network create $BUILD_TAG'
                sh 'docker run -d --name $DOCKER_IMAGE_RABBITMQ --rm --network $BUILD_TAG $DOCKER_ID/$DOCKER_IMAGE_RABBITMQ:$DOCKER_TAG'
                sh 'docker stop $DOCKER_IMAGE_RABBITMQ'
                sh 'docker network rm $BUILD_TAG'    
                }
        }
        stage('Push') {
            environment {
                DOCKER_PASS = credentials("DOCKER_HUB_PASS")
            }
         steps {
            sh 'docker image tag $DOCKER_ID/$DOCKER_IMAGE_RABBITMQ:$DOCKER_TAG $DOCKER_ID/$DOCKER_IMAGE_RABBITMQ:latest'
            sh 'docker login -u $DOCKER_ID -p $DOCKER_PASS'
            sh 'docker push $DOCKER_ID/$DOCKER_IMAGE_RABBITMQ:$DOCKER_TAG && docker push $DOCKER_ID/$DOCKER_IMAGE_RABBITMQ:latest'
            }
        }
        stage('Deploy EKS') {
            environment {
                KUBECONFIG = credentials("EKS_CONFIG")  
                AWSKEY = credentials("AWS_KEY")
                AWSSECRETKEY = credentials("AWS_SECRET_KEY")
                AWSREGION = credentials("AWS_REGION")
                EKSCLUSTERNAME = credentials("EKS_CLUSTER")

            }
            steps {
                sh 'rm -Rf .kube'
                sh 'mkdir .kube'
                sh 'touch .kube/config'
                sh 'sudo chmod 777 .kube/config'
                sh 'rm -Rf .aws'
                sh 'mkdir .aws'
                sh 'aws configure set aws_access_key_id $AWSKEY'
                sh 'aws configure set aws_secret_access_key $AWSSECRETKEY'
                sh 'aws configure set region $AWSREGION'
                sh 'aws configure set output text'                
                sh 'aws eks --region $AWSREGION update-kubeconfig --name $EKSCLUSTERNAME --kubeconfig .kube/config'
                sh 'aws eks list-clusters'
                sh 'kubectl cluster-info --kubeconfig .kube/config'
                sh 'kubectl apply -f ./manifests -n $NAMESPACE --kubeconfig .kube/config'
            }
        }
    }
}