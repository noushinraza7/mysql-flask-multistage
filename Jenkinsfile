pipeline{
    agent any
    stages{
        stage("git"){
            steps{
                git branch: "main"
                url: https://github.com/noushinraza7/mysql-flask-multistage.git
            }
        }
    }
}