pipeline {
    agent {
        label 'dev'
    }
         tools{
            go 'go'
         }
        environment {
        AWS_REGION      = "us-east-1"
        AWS_ACCOUNT_ID  = "985059095589"
        ECR_REPO_NAME   = "gitops-agrocd-frontend"
        IMAGE_TAG       = "${BUILD_NUMBER}"
        ECR_REGISTRY    = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com"
        IMAGE_NAME      = "${ECR_REGISTRY}/${ECR_REPO_NAME}:${IMAGE_TAG}"
    }


    stages {

        stage('Git Checkout') {
            steps {
                git url: 'https://github.com/ITkannadigaru/frontend.git', branch: 'main'
            }
        }

        stage('Run Unit Tests') {
            steps {
                sh 'go test ./...'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    printenv
                    docker build -t ${IMAGE_NAME} .
                '''
            }
        }
       stage('Load AWS Secrets') {
            steps {
                script {
                    def secretJson = sh(
                        script: '''
                            aws secretsmanager get-secret-value \
                                --secret-id awslogin \
                                --region ${AWS_REGION} \
                                --query SecretString \
                                --output text
                        ''',
                        returnStdout: true
                    ).trim()

                    def secrets = new groovy.json.JsonSlurperClassic().parseText(secretJson)
                    env.AWS_ACCESS_KEY_ID = secrets['access-id']
                    env.AWS_SECRET_ACCESS_KEY = secrets['secret']
                }
            }
        }
        stage('ECR Login') {
            steps {
                sh '''
                       aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }
        stage('Push to ecr') {
            steps {
                sh "docker push ${IMAGE_NAME}"
            }
        }
    }
}
