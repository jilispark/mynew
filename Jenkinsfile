pipeline {
    agent { 
        node {
            label 'slave01'
        }
    }
    
    environment {
        def APP_NAME = "nodejs-app-dev"
        def envName = "dev"
        def k8sNamespace = "dev"
        def eksCluster = "sample-eks"
        def s3secretbucket = "configmap-variables-prince"
        def s3configbucket = "configmap-variables-prince"
        def region = "us-east-1"
        def AWS_ACCOUNT = "897585983198"
        def CONTAINER = "sample-nodejs"
        def ARGOCD_SERVER= "a124d6af24d234dd5889feb1735904cb-1188587285.us-east-1.elb.amazonaws.com"        
    }

    stages {       
        stage('Prepare') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: '*/master']], 
                extensions: [], 
                userRemoteConfigs: [[
                    url: 'https://github.com/princejoseph4043/nodejs_nginx_app_std.git']]
                ])

            }
        }
    
        stage ('Docker_Build') {
            steps {
                sh '''
                cd docker_nodejs_nginx
                docker build -t ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:v1-${BUILD_ID} .
                '''                
            }
        }

        stage ('Deploy image to ECR') {
             steps {
                    sh '''
                    $(aws ecr get-login --region ${region} --no-include-email)
#                    docker tag ${CONTAINER}:v1-${BUILD_ID} ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:v-${BUILD_ID}
                    docker push ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:v1-${BUILD_ID}
#                    docker tag ${CONTAINER}-v-${BUILD_ID} ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:latest
#                    docker push ${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:latest
                    '''
               }    
            }

        stage ('Get kuberntes ConfigMap from s3') {
            steps {
                sh '''
                pwd
                ls -l        
                aws s3 cp s3://${s3secretbucket}/default.conf .
                aws s3 cp s3://${s3secretbucket}/local_env.yaml .
                ls -l
                aws eks --region ${region} update-kubeconfig --name ${eksCluster}
                
                kubectl create cm app-properties-v1-${BUILD_ID} --from-file=local_env.yaml -n ${k8sNamespace}
                kubectl create cm nginx-config-v1-${BUILD_ID} --from-file=default.conf -n ${k8sNamespace}
                aws s3 cp local_env.yaml s3://${s3configbucket}/${APP_NAME}/${BUILD_ID}/local_env.yaml
                rm -f local_env.yaml
                rm -f default.conf                                     
                '''              
            }
        }

        stage('Checkout k8s files') {
            steps {
                checkout([$class: 'GitSCM', 
                branches: [[name: '*/master']], 
                extensions: [], 
                userRemoteConfigs: [[credentialsId: '5fe89fbd-eece-401e-985f-9ddeaeaeb76a', 
                url: 'git@github.com:princejoseph4043/argocd_kube_deploy_new.git']]])        
            }
        }

        stage ('Commit to Git Repo') {
            steps {
                sh '''
                pwd
                ls -l
                cd $WORKSPACE
                cat deployment-Service.yml
                git config --global user.email "princejoseph4043@gmail.com"
                git config --global user.name "Prince Joseph"                
                git checkout master
                git pull origin master
                git branch
                cat deployment-Service.yml                                               
                sed -i "s|<IMAGE-ID>|${AWS_ACCOUNT}.dkr.ecr.${region}.amazonaws.com/${CONTAINER}:v-${BUILD_ID}|g" deployment-Service.yml
                sed -i "s|<CONFIGMAP-ID>|nginx-config-v1-${BUILD_ID}|g" deployment-Service.yml
                cat deployment-Service.yml
                                                      
                # Automatic Deploy to ArgoCD
#               ARGOCD_SERVER=${ARGOCD_SERVER} argocd --grpc-web app sync ${APP_NAME} --force
#               ARGOCD_SERVER=${ARGOCD_SERVER} argocd --grpc-web app wait ${APP_NAME} --timeout 600
                '''
               }
            }   


        }                  

    }

}