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

        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    sudo rm -rf /mnt/web-server/apache-tomcat-10.1.60/webapps/LoginWebApp
                    sudo rm -f /mnt/web-server/apache-tomcat-10.1.60/webapps/LoginWebApp.war

                    sudo cp /mnt/project/target/LoginWebApp.war \
                    /mnt/web-server/apache-tomcat-10.1.60/webapps/
                '''
            }
        }

        stage('Restart Tomcat') {
            steps {
                sh '''
                    sudo //mnt/web-server/apache-tomcat-10.1.60/bin/shutdown.sh || true
                    sleep 10
                    sudo /mnt/web-server/apache-tomcat-10.1.60/bin/startup.sh
                '''
            }
        }

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

                        sed -i "s|DB_USERNAME|${DB_USERNAME}|g" userRegistration.jsp
                        sed -i "s|DB_PASSWORD|${DB_PASSWORD}|g" userRegistration.jsp

                        echo "Application database configuration completed."
                    '''
                }
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
                          -h ${DB_HOST} \
                          -u ${DB_USERNAME} \
                          -p${DB_PASSWORD} \
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
                        echo "Checking USER table in RDS..."

                        mysql \
                          -h ${DB_HOST} \
                          -u ${DB_USERNAME} \
                          -p${DB_PASSWORD} \
                          ${DB_NAME} \
                          -e "SHOW TABLES; SELECT * FROM USER;"
                    '''
                }
            }
        }
    }



    }
}
