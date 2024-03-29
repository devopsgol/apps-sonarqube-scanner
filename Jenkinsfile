pipeline {
    agent {
        label 'slave-devopsgol'
    }
    
    tools {
        maven 'maven3'
    }

    stages {
        stage("checkout"){
            steps{
                checkout scm
                sh 'mvn clean install'
                echo 'Git Checkout Completed'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonarqube-devops') {
                    sh 'mvn clean package'
                    sh ''' mvn clean verify sonar:sonar -Dsonar.projectKey=riset-sonar-devopsgol -Dsonar.projectName='riset-sonar-devopsgol' -Dsonar.host.url=http://10.20.40.45:9010 '''
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
                    sh 'sudo docker build -t test-scanner-with-soanrqube .'
                    echo 'Build Docker Image Completed'
                }
            }
        }

        stage('Docker Push') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'docker_cred', passwordVariable: 'DOCKERHUB_PASSWORD', usernameVariable: 'DOCKERHUB_USERNAME')]) {
                        sh ''' sudo docker login -u $DOCKERHUB_USERNAME -p $DOCKERHUB_PASSWORD" '''
                        sh ''' sudo docker tag test-scanner-with-soanrqube:latest adinugroho251/test-scanner-with-soanrqube:latest" '''
                    }
                    sh 'sudo docker push adinugroho251/test-scanner-with-soanrqube:latest'
                }
            }
        }

        stage ('Docker Run') {
            steps {
                script {
                    sh 'sudo docker run -d -p 8099:8080 --name deploy-test-scanner-with-soanrqube-test-successfully  adinugroho251/test-scanner-with-soanrqube:latest'
                    echo 'Docker Run Completed'
                }
            }
        }

    }
}
