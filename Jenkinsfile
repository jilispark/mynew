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
        def timestamp = "date "+%Y%m%d%H%M""
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
                docker build -t sample-nodejs:v-${BUILD_ID} .
                '''                
            }
        }

        stage ('Deploy image to ECR') {
             steps {
                    sh '''
                    $(aws ecr get-login --region ${region} --no-include-email)
                    docker tag ${CONTAINER}:v-${BUILD_ID} ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:v-${BUILD_ID}
                    docker push ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:v-${BUILD_ID}
#############       docker tag ${CONTAINER}:latest ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:latest
#############       docker push ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:latest
                    '''
               }    
            }

        stage ('Get kuberntes ConfigMap from s3') {
            steps {
                sh '''
                pwd
                ls -l  
                
                echo "current time is ${timestamp}"         
                aws s3 cp s3://${s3secretbucket}/default.conf .
                aws s3 cp s3://${s3secretbucket}/local_env.yaml .
                ls -l
                aws eks --region ${region} update-kubeconfig --name ${eksCluster}
                
                kubectl create cm app-properties-${BUILD_ID} --from-file=local_env.yaml -n ${k8sNamespace}
                kubectl create cm nginx-config-${BUILD_ID} --from-file=default.conf -n ${k8sNamespace}
            #    aws s3 cp local_env.yaml s3://${s3ConfigBucket}/${appname}/${env.BUILD_ID}                    
                '''               
            }
        }

        }               

    }