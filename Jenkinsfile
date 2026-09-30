pipeline {

    agent {
        label {
            label 'built-in'
            customWorkspace '/mnt/project/'
        }
    }

    environment {
        TOMCAT_HOME = '/mnt/web-server/apache-tomcat-10.1.60'
        APP_NAME    = 'LoginWebApp'
        WAR_NAME    = 'LoginWebApp.war'

        DB_HOST     = 'velocity-db.c502c4e2yh9e.ap-south-1.rds.amazonaws.com'
        DB_NAME     = 'test'
    }

    tools {
        maven 'Maven_auto'
    }

    stages {

        stage('Configure RDS Credentials') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'rds-db-credentials',
                        usernameVariable: 'DB_USERNAME',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "Configuring application database credentials..."

                        sed -i "s|DB_USERNAME|${DB_USERNAME}|g" \
                        src/main/webapp/userRegistration.jsp

                        sed -i "s|DB_PASSWORD|${DB_PASSWORD}|g" \
                        src/main/webapp/userRegistration.jsp

                        echo "RDS credentials configured."
                    '''
                }
            }
        }

        stage('Build') {
            steps {

                sh '''
                    mvn clean package

                    echo "WAR created:"
                    ls -lh target/LoginWebApp.war
                '''
            }
        }

        stage('Configure RDS Database') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'rds-db-credentials',
                        usernameVariable: 'DB_USERNAME',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "Connecting to AWS RDS..."

                        mysql \
                          -h "${DB_HOST}" \
                          -u "${DB_USERNAME}" \
                          -p"${DB_PASSWORD}" \
                          -e "
                            CREATE DATABASE IF NOT EXISTS ${DB_NAME};

                            USE ${DB_NAME};

                            CREATE TABLE IF NOT EXISTS USER (
                                id INT AUTO_INCREMENT PRIMARY KEY,
                                first_name VARCHAR(50),
                                last_name VARCHAR(50),
                                email VARCHAR(100),
                                username VARCHAR(50),
                                password VARCHAR(255),
                                regdate DATE
                            );

                            SHOW TABLES;

                            DESCRIBE USER;
                          "

                        echo "RDS database configuration completed."
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {

                sh '''
                    echo "Stopping/removing old application..."

                    sudo rm -rf ${TOMCAT_HOME}/webapps/${APP_NAME}
                    sudo rm -f ${TOMCAT_HOME}/webapps/${WAR_NAME}

                    echo "Deploying new WAR..."

                    sudo cp target/${WAR_NAME} \
                    ${TOMCAT_HOME}/webapps/

                    echo "WAR deployed:"
                    ls -lh ${TOMCAT_HOME}/webapps/${WAR_NAME}
                '''
            }
        }

        stage('Restart Tomcat') {
            steps {

                sh '''
                    echo "Stopping Tomcat..."

                    sudo ${TOMCAT_HOME}/bin/shutdown.sh || true

                    sleep 10

                    echo "Starting Tomcat..."

                    sudo ${TOMCAT_HOME}/bin/startup.sh

                    sleep 10

                    echo "Tomcat process:"

                    ps -ef | grep '[t]omcat' || true
                '''
            }
        }

        stage('Verify Application') {
            steps {

                sh '''
                    echo "Testing LoginWebApp..."

                    curl -I --max-time 15 \
                    http://localhost:8080/${APP_NAME}/ || true
                '''
            }
        }

        stage('Verify RDS Data') {
            steps {

                withCredentials([
                    usernamePassword(
                        credentialsId: 'rds-db-credentials',
                        usernameVariable: 'DB_USERNAME',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {

                    sh '''
                        echo "Checking RDS USER table..."

                        mysql \
                          -h "${DB_HOST}" \
                          -u "${DB_USERNAME}" \
                          -p"${DB_PASSWORD}" \
                          "${DB_NAME}" \
                          -e "SHOW TABLES; SELECT * FROM USER;"
                    '''
                }
            }
        }
    }

    post {

        success {
            echo '''
========================================
 DEPLOYMENT SUCCESSFUL
========================================
 Application : LoginWebApp
 Tomcat      : 8080
 Database    : AWS RDS MySQL
 Database    : test
 Table       : USER
========================================
'''
        }

        failure {
            echo '''
========================================
 DEPLOYMENT FAILED
========================================
Check the Jenkins Console Output.
========================================
'''
        }
    }
}
