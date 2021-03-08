///////////Variables///////////


pipeline {
    agent { 
        node {
            label 'slave01'
        }
    }
    
    environment {
        def appName = "nodejsnginx"
        def envName = "dev"
        def k8sNamespace = "dev"
        def eksCluster = "sample-eks"
        def s3secretbucket = "configmap-variables-prince"
        def s3configbucket = "configmap-variables-prince"
        def region = "us-east-1"
        def AWS_ACCOUNT = "897585983198"
        def CONTAINER = "sample-nodejs"
    }

    stages {       
        stage('Prepare') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: '*/master']], 
                extensions: [], 
                userRemoteConfigs: [[
                    url: 'https://github.com/princejoseph4043/nodejs_nginx_app.git']]
                ])

            }
        }
    
        stage ('Docker_Build') {
            steps {
                sh '''
                cd docker_nodejs_nginx
                docker build -t sample-nodejs:v-${env.region}-${BUILD_ID} .
                '''                
            }
        }

        stage ('Deploy image to ECR') {
             steps {
                    sh '''
                    $(aws ecr get-login --region ${env.region} --no-include-email)
                    docker tag ${CONTAINER}:v-${BUILD_ID} ${AWS_ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/${CONTAINER}:v-${BUILD_ID}
                    docker push ${AWS_ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/${CONTAINER}:v-${BUILD_ID}
#############       docker tag ${CONTAINER}:latest ${AWS_ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/${CONTAINER}:latest
#############       docker push ${AWS_ACCOUNT}.dkr.ecr.${REGION}.amazonaws.com/${CONTAINER}:latest
                    '''
               }    
            }

        stage ('Get kuberntes ConfigMap from s3') {
            steps {
                sh '''
                pwd
                ls -l           
                aws s3 cp s3://configmap-variables-prince/default.conf .
                aws s3 cp s3://configmap-variables-prince/local_env.yaml .
                ls -l
                aws eks --region us-east-1 update-kubeconfig --name sample-eks
                
                kubectl create cm app-properties-${BUILD_ID} --from-file=local_env.yaml -n dev
                kubectl create cm nginx-config-${BUILD_ID} --from-file=default.conf -n dev
                aws s3 cp local_env.yaml s3://${s3ConfigBucket}/${appname}/${env.BUILD_ID}                    
                '''               
            }
        }

        }               

    }