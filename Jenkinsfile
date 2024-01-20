pipeline {
    agent {
        label 'slave-devopsgol'
    }
    
    tools {
        maven 'maven3'
    }

    stages {
        stage('Git Checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/devopsgol/apps-sonarqube-scanner/']])
                bat 'mvn clean install'
                echo 'Git Checkout Completed'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-devops') {
                    bat 'mvn clean package'
                    bat ''' mvn clean verify sonar:sonar -Dsonar.projectKey=riset-sonar-devopsgol -Dsonar.projectName='riset-sonar-devopsgol' -Dsonar.host.url=http://10.20.40.45:9010 '''
                    echo 'SonarQube Analysis Completed'
                }
            }
        }
        stage("Quality Gate") {
            steps {
                waitForQualityGate abortPipeline: true
                echo 'Quality Gate Completed'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    bat 'docker build -t test-scanner-with-soanrqube .'
                    echo 'Build Docker Image Completed'
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_cred', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        bat ''' sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD" '''
                        bat ''' sudo docker tag test-scanner-with-soanrqube:latest adinugroho251/test-scanner-with-soanrqube:latest" '''
                    }
                    bat 'sudo docker push adinugroho251/test-scanner-with-soanrqube:latest'
                }
            }
        }

        stage ('Docker Run') {
            steps {
                script {
                    bat 'sudo docker run -d -p 8099:8080 --name deploy-test-scanner-with-soanrqube-test-successfully  adinugroho251/test-scanner-with-soanrqube:latest'
                    echo 'Docker Run Completed'
                }
            }
        }

    }
    post {
        always {
            bat 'docker logout'
        }
    }
}
