pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_REPO  = 'docker.io/mohdanaskhan451'
        APP_NAME     = 'ci-cd'
    }

    stages {
        stage('Clean Workspace') {
            steps { cleanWs() }
        }

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Mohammedanaskhan7/ci-cd.git'
            }
        }

        stage('Trivy Scan - Code') {
            steps {
                sh "trivy fs --exit-code 0 --severity HIGH,CRITICAL ."
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -Dmaven.test.skip=true'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Petclinic \
                        -Dsonar.java.binaries=. \
                        -Dsonar.projectKey=Petclinic
                    '''
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImage = "${env.DOCKER_REPO}/${env.APP_NAME}"
                    def dockerTag = "${dockerImage}:${env.BUILD_NUMBER}"
                    withCredentials([string(credentialsId: 'Docker-Passwd', variable: 'DOCKER_PASSWD')]) {
                        sh "echo ${DOCKER_PASSWD} | docker login -u mohdanaskhan451 --password-stdin"
                        sh "docker build -t ${dockerTag} ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def dockerImage = "${env.DOCKER_REPO}/${env.APP_NAME}"
                    def dockerTag = "${dockerImage}:${env.BUILD_NUMBER}"
                    withCredentials([string(credentialsId: 'Docker-Passwd', variable: 'DOCKER_PASSWD')]) {
                        sh "echo ${DOCKER_PASSWD} | docker login -u mohdanaskhan451 --password-stdin"
                        sh "docker push ${dockerTag}"
                    }
                }
            }
        }

        stage('Trivy Scan - Image') {
            steps {
                script {
                    def dockerImage = "${env.DOCKER_REPO}/${env.APP_NAME}"
                    def dockerTag = "${dockerImage}:${env.BUILD_NUMBER}"
                    sh "trivy image ${dockerTag}"
                }
            }
        }

        stage('Deploy Application') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'Docker-Passwd', variable: 'DOCKER_PASSWD')]) {
                        sh '''
                            echo ${DOCKER_PASSWD} | docker login -u mohdanaskhan451 --password-stdin
                            echo TAG=${env.BUILD_NUMBER} > .env
                            docker-compose -f docker-compose.yaml down || true
                            docker-compose -f docker-compose.yaml up -d
                        '''
                    }
                }
            }
        }
    }

    post {
        always {
            script {
                def dockerImage = "${env.DOCKER_REPO}/${env.APP_NAME}"
                def dockerTag = "${dockerImage}:${env.BUILD_NUMBER}"
                sh "docker rmi -f ${dockerTag} || true"
            }
        }
    }
}
