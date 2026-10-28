
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'flask-mysql-app'
        NETWORK_NAME = 'flask-network'
        DB_CONTAINER = 'flask-mysql-db'
        APP_CONTAINER = 'flask-app'
        DB_VOLUME = 'flask-mysql-data'
        APP_PORT = '5000'
        DB_NAME = 'flask_db'
        DB_USER = 'appuser'
        DB_PASSWORD = 'apppass123'
        DB_ROOT_PASSWORD = 'rootpass123'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/noushinraza7/mysql-flask-multistage.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Prepare Network and Volume') {
            steps {
                sh '''
                    docker network inspect ${NETWORK_NAME} >/dev/null 2>&1 ||
                        docker network create ${NETWORK_NAME}

                    docker volume inspect ${DB_VOLUME} >/dev/null 2>&1 ||
                        docker volume create ${DB_VOLUME}
                '''
            }
        }

        stage('Deploy MySQL') {
            steps {
                sh '''
                    docker rm -f ${DB_CONTAINER} || true

                    docker run -d \
                        --name ${DB_CONTAINER} \
                        --network ${NETWORK_NAME} \
                        --network-alias mysql \
                        --restart unless-stopped \
                        --health-cmd="mysqladmin ping -h localhost -uroot -p${DB_ROOT_PASSWORD}" \
                        --health-interval=5s \
                        --health-timeout=5s \
                        --health-retries=20 \
                        -e MYSQL_ROOT_PASSWORD=${DB_ROOT_PASSWORD} \
                        -e MYSQL_DATABASE=${DB_NAME} \
                        -e MYSQL_USER=${DB_USER} \
                        -e MYSQL_PASSWORD=${DB_PASSWORD} \
                        -v ${DB_VOLUME}:/var/lib/mysql \
                        mysql:8.4

                    for i in $(seq 1 60); do
                        STATUS=$(docker inspect \
                            --format='{{.State.Health.Status}}' \
                            ${DB_CONTAINER})

                        if [ "$STATUS" = "healthy" ]; then
                            echo "MySQL is ready"
                            exit 0
                        fi

                        sleep 2
                    done

                    docker logs ${DB_CONTAINER}
                    echo "MySQL failed to become healthy"
                    exit 1
                '''
            }
        }

        stage('Deploy Flask') {
            steps {
                sh '''
                    docker rm -f ${APP_CONTAINER} || true

                    docker run -d \
                        --name ${APP_CONTAINER} \
                        --network ${NETWORK_NAME} \
                        --restart unless-stopped \
                        -p ${APP_PORT}:5000 \
                        -e DB_HOST=mysql \
                        -e DB_USER=${DB_USER} \
                        -e DB_PASSWORD=${DB_PASSWORD} \
                        -e DB_NAME=${DB_NAME} \
                        ${IMAGE_NAME}:${BUILD_NUMBER}

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'Flask and MySQL deployment completed!'
        }

        failure {
            echo 'Pipeline failed. Check Console Output.'
        }
    }
}
