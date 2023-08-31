pipeline {

    agent any // Run on any agent (environment) default is genkins

    tools {
        maven 'my-maven'
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql')
    }
    stages {

        stage ('Build with maven') {
            steps {
                sh 'mvn --version'
                sh 'java --version'
                // sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }
        
        stage ('Package/Pushing image') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url:'https://index.docker.io/v2') {
                    sh 'docker build -t phamvant/spring-test .'
                    sh 'docker push phamvant/spring-test'
                }
            }
        }

        stage ('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql'
                sh 'docker network create || echo "this network existed"'
                sh 'docker container stop thuan-mysql || echo "this container not exist'
                sh 'echo y | docker container prune'

                sh "docker run --name thuan-mysql --rm --network dev -e thuan-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example -d mysql:8.0"
                sh 'sleep 20'
                sh 'docker exec -i thuan-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script'
            }
        }

        stage ('Deloy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull phamvant/spring-test'
                sh 'docker network create || echo "this network existed"'
                sh 'docker container stop thuan-spring || echo "this container not exist"'
                sh 'echo y | docker container prune'

                sh "docker container run -d --rm --name thuan-spring -p 8081:8080 --network dev phamvant/spring-test"
            }
        }
   } 

    post {
        //Clean after build
        always {
            cleanWs()
        }
    }
}