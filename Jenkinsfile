pipeline{
    agent any
    stages{
        stage("git"){
            steps{
                git branch: "main",
                    url: 'https://github.com/noushinraza7/mysql-flask-multistage.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t flask-app .
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                docker-compose down --remove-orphans || true
                docker-compose up -d
                docker-compose ps
                '''
            }
        }

    }
}