
pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/noushinraza7/mysql-flask-multistage.git'
        }

        stage('Validate Files') {
            steps {
                sh '''
                    test -f app.py
                    test -f Dockerfile
                    test -f docker-compose.yml
                    test -f requirements.txt
                    echo "All required files are present"
                '''
            }
        }

        stage('Validate Compose') {
            steps {
                sh 'docker compose config -q'
            }
        }

        stage('Build Image') {
            steps {
                sh 'docker build -t flask-app .'
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker compose down --remove-orphans || true
                    docker compose up -d
                    docker compose ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the stage logs.'
        }
    }
}
