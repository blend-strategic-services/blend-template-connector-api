pipeline {
    agent any
    tools {
        maven 'maven-3.6.3'
    }
    stages {
        stage('Unit Test') {
            steps {
                script {
                    setEnvironmentVars(params.environment)
                }
                slackSend (color: '#FFFF00', message: "${env.ENV} > STARTED : Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (<${env.BUILD_URL}|Open>)")
                slackSend (color: '#FFFF00', message: "${env.ENV} > Executing Tests: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (<${env.BUILD_URL}|Open>)")
                echo 'Running tests...'
                sh 'mvn -Denv=test clean test'
            }
        }
        stage('Deploy to Binary Repo') {
            when {
                anyOf {
                    expression { params.environment == "Production" };
                }
            }
            steps {
                slackSend (color: '#FFFF00', message: "${env.ENV} > Deploying to Nexus: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (<${env.BUILD_URL}|Open>)")
                echo 'Deploy to Binary Repo ' + "${params.environment}"
                sh 'mvn -DskipTests deploy'
            }
        }
        stage('Deploy CloudHub') {
            environment {
                ANYPOINT_CREDENTIALS = credentials('anypoint.credentials')
            }
            steps {
                sh 'echo $PATH'
                slackSend (color: '#FFFF00', message: "${env.ENV} > Deploying to Cloudhub: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (<${env.BUILD_URL}|Open>)")
                sh 'mvn deploy -DskipTests -DmuleDeploy -Dmule.version=' + "${env.VERSION}" + ' -Danypoint.username=${ANYPOINT_CREDENTIALS_USR} -Danypoint.password=${ANYPOINT_CREDENTIALS_PSW} -e'
            }
        }
    }
    post {
        failure {
            slackSend color: 'danger', message: "@here \n${env.ENV} > FAILED ${env.JOB_NAME} [${env.BUILD_NUMBER}] (<${env.BUILD_URL}|Open>)"
        }
        success {
            slackSend color: 'good', message: "${env.ENV} > SUCCESS ${env.JOB_NAME} [${env.BUILD_NUMBER}] (<${env.BUILD_URL}|Open>)"
        }
    }
}

void setEnvironmentVars(String envParam) {
    if (envParam != null){
        echo "DEBUG: envParam=$envParam"

        env.CLOUDHUB_ENVIRONMENT = envParam

        if (env.CLOUDHUB_ENVIRONMENT == "Development") {
            env.ENV = "dev"
            env.VERSION = "4.3.0"              
            env.CLOUDHUB_WORKERTYPE = "Micro"
            env.CLOUDHUB_WORKERS = "1"
        } else if (env.CLOUDHUB_ENVIRONMENT == "Beta") {
            env.ENV = "beta"
            env.VERSION = "4.3.0"              
            env.CLOUDHUB_WORKERTYPE = "Micro"
            env.CLOUDHUB_WORKERS = "1"
        } else if (env.CLOUDHUB_ENVIRONMENT == "Production") {
            env.ENV = "prod"
            env.VERSION = "4.3.0"              
            env.CLOUDHUB_WORKERTYPE = "Micro"
            env.CLOUDHUB_WORKERS = "1"
        }
    } else {
        env.CLOUDHUB_ENVIRONMENT = "Development"
        env.ENV = "dev"
        env.VERSION = "4.3.0"          
        env.CLOUDHUB_WORKERTYPE = "Micro"
        env.CLOUDHUB_WORKERS = "1"
    }

    withCredentials([usernamePassword(credentialsId: 'anypoint.client.' + env.ENV, passwordVariable: 'ANYPOINT_CLIENT_SECRET', usernameVariable: 'ANYPOINT_CLIENT_ID')]) {
        env.ANYPOINT_CLIENT_ID = "$ANYPOINT_CLIENT_ID"
        env.ANYPOINT_CLIENT_SECRET = "$ANYPOINT_CLIENT_SECRET"
    }
	withCredentials([usernamePassword(credentialsId: 'logger', passwordVariable: 'LOGGER_TOKEN', usernameVariable: 'LOGGER_URL')]) {
        env.LOGGER_URL = "$LOGGER_URL"
        env.LOGGER_TOKEN = "$LOGGER_TOKEN"
    }
    withCredentials([string(credentialsId: env.ENV + '.secretKey', variable: 'MASTER_KEY')]) {
        env.MASTER_KEY = "$MASTER_KEY"
    }
    withCredentials([usernamePassword(credentialsId: 'cloudhub.notification', passwordVariable: 'CLOUDHUB_NOTIFICATION_PASSWORD', usernameVariable: 'CLOUDHUB_NOTIFICATION_USERNAME')]) {
        env.CLOUDHUB_NOTIFICATION_USERNAME = "$CLOUDHUB_NOTIFICATION_USERNAME"
        env.CLOUDHUB_NOTIFICATION_PASSWORD = "$CLOUDHUB_NOTIFICATION_PASSWORD"
    }

    echo "DEBUG: setEnvironment: CLOUDHUB_ENVIRONMENT: ${env.CLOUDHUB_ENVIRONMENT}"
    echo "DEBUG: setEnvironment: ENV: ${env.ENV}"
    echo "DEBUG: setEnvironment: PATH: ${env.PATH}"
}
