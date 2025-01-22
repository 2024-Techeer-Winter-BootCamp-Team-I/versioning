pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose-deploy.yml'
        DEPLOY_SERVER="jinkitea@34.64.246.46"
    }

    stages {
        stage('Checkout') {
            steps {
		            deleteDir()
                git branch: 'main', url: 'https://github.com/2024-Techeer-Winter-BootCamp-Team-I/versioning.git'
            }
        }

        stage('Test') {
            steps {
                script {
                    sh "docker --version"
                    sh "docker compose --version"
                }
            }
        }

        stage('Deploy') {
            when {
                anyOf {
                    branch 'main'
                    branch 'master'
                }
            }
            steps {
                script {
                    sshagent(['deploy-server-access']) {
                        sh """
                        scp -r -o StrictHostKeyChecking=no . ${DEPLOY_SERVER}:~/directory
                        ssh -o StrictHostKeyChecking=no ${DEPLOY_SERVER} '
                        cd ~ &&
                        ls -al &&
                        docker compose -f ${DOCKER_COMPOSE_FILE} down --remove-orphans
                        docker compose -f ${DOCKER_COMPOSE_FILE} pull &&
                        docker compose -f ${DOCKER_COMPOSE_FILE} up -d'
                        """
                    }
                }
            }
        }
    }

    post {
		    always {
                cleanWs(cleanWhenNotBuilt: false,
                        deleteDirs: true,
                        disableDeferredWipeout: true,
                        notFailBuild: true,
                        patterns: [[pattern: '.gitignore', type: 'INCLUDE'],
                                [pattern: '.propsfile', type: 'EXCLUDE']])
        }
        success {
            echo 'Build and deployment successful!'
            //slackSend message: "Service deployed successfully - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
        failure {
            echo 'Build or deployment failed.'
            //slackSend failOnError: true, message: "Build failed  - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)"
        }
    }
}