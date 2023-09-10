pipeline {
    agent any
    stages {
        stage('build da imagem docker') {
            steps {
                sh 'docker build -t devops/app .'
            }
        }

        stage('subir docker compose - redis e app') {
            steps {
                sh 'docker-compose up --build -d'
            }
        }

        stage('sleep para subida de containers') {
            steps {
                sh 'sleep 10'
            }
        }

        stage('sonarqube validation') {
            steps {
                def scannerHome = tool 'SonarScanner'
                withSonarQubeEnv() {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('teste da aplicação') {
            steps {
                sh 'chmod +x teste-app.sh'
                sh './teste-app.sh'
            }
        }

        stage('shutdown dos containers de teste') {
            steps {
                sh 'docker compose down'
            }
        }

        stage('upload docker image') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'nexus-user', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                        sh 'docker login -u $USERNAME -p $PASSWORD ${NEXUS_URL}'
                        sh 'docker tag devops/app:latest ${NEXUS_URL}/devops/app'
                        sh 'docker push ${NEXUS_URL}/devops/app'
                    }
                }
            }
        }

        stage('Apply k8s files') {
            steps {
                sh '/usr/local/bin/kubectl apply -f ./k3s/redis.yaml'
                sh '/usr/local/bin/kubectl apply -f ./k3s/redis-app.yaml'
            }
        }
    }
}
