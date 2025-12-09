pipeline {
    agent any

    environment {
        NODEJS_HOME = tool name: 'node18'
        PATH = "${NODEJS_HOME}/bin:${env.PATH}"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo 'Pulling code from GitHub...'
                git branch: 'main',
                    credentialsId: 'github-token',
                    url: 'https://github.com/narapureddy90/mini-project1.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing dependencies...'
                sh '''
                    cd server && npm install
                    cd ../client && npm install
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis...'
                withSonarQubeEnv('local-sonar') {
                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=myproject \
                        -Dsonar.sources=./server \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=$SONAR_AUTH_TOKEN
                    '''
                }
            }
        }

        stage('Build Project') {
            steps {
                echo 'Building frontend...'
                sh '''
                    cd client
                    npm run build
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image...'
                sh '''
                    docker build -t my-fullstack-app:latest .
                '''
            }
        }

        stage('Run Container') {
            steps {
                echo 'Running Docker container...'
                sh '''
                    docker stop myapp || true
                    docker rm myapp || true
                    docker run -d --name myapp -p 5000:5000 my-fullstack-app:latest
                '''
            }
        }

        stage('Archive Artifacts') {
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: 'client/build/**', fingerprint: true
                archiveArtifacts artifacts: 'server/**/*', fingerprint: true
            }
        }
    }

    post {
        success {
            echo "✔ Pipeline Completed Successfully!"
        }
        failure {
            echo "❌ Pipeline Failed. Check logs."
        }
    }
}
