///////////Variables///////////

def appName = "nodejsnginx"
def envName = "dev"
def k8sNamespace = "dev"
def eksCluster = "sample-eks"
def s3secretbucket = "configmap-variables-prince"
def s3configbucket = "configmap-variables-prince"
def REGION = "us-east-1"
def AWS_ACCOUNT = "897585983198"
def CONTAINER = "sample-nodejs"

pipeline {

    agent {
        node {
            label 'slave01'
        }
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
                sh '''cd docker_nodejs_nginx
                docker build -t sample-nodejs:v-${BUILD_ID} .
                '''                
            }
        }

        stage ("Print variable") {
      steps {
        echo "My variable is ${eksCluster}"
      }
    }

        stage ('Deploy image to ECR') {
             steps {
                    sh "
                    echo my env is ${envName}
                    echo my region is ${REGION}
                    "
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