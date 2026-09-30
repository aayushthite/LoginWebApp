pipeline {

    agent any

    tools {
        maven 'Maven_auto'
    }

    environment {
        PROJECT_DIR = '/mnt/project'
        TOMCAT_HOME = '/mnt/web-server/apache-tomcat-10.1.60'

        RDS_HOST = 'velocity-db.c502c4e2yh9e9.ap-south-1.rds.amazonaws.com'
        DB_NAME  = 'test'
    }

    stages {

        stage('Configure RDS Credentials') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'rds-db-credentials',
                        usernameVariable: 'RDS_USER',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "========================================"
                        echo "Configuring application database credentials"
                        echo "========================================"

                        sed -i "s|DB_USERNAME|${RDS_USER}|g" \
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
                    echo "========================================"
                    echo "Building WAR"
                    echo "========================================"

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
                        usernameVariable: 'RDS_USER',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "========================================"
                        echo "Connecting to AWS RDS"
                        echo "========================================"

                        mariadb \
                            --skip-ssl-verify-server-cert \
                            -h "$RDS_HOST" \
                            -u "$RDS_USER" \
                            -p"$DB_PASSWORD" \
                            -e "
                                CREATE DATABASE IF NOT EXISTS $DB_NAME;

                                USE $DB_NAME;

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

                        echo "RDS database configuration completed successfully."
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Deploying WAR to Tomcat"
                    echo "========================================"

                    cp -f target/LoginWebApp.war \
                        "$TOMCAT_HOME/webapps/"

                    echo "WAR deployed successfully."

                    ls -lh "$TOMCAT_HOME/webapps/LoginWebApp.war"
                '''
            }
        }

        stage('Restart Tomcat') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Restarting Tomcat"
                    echo "========================================"

                    "$TOMCAT_HOME/bin/shutdown.sh" || true

                    sleep 5

                    if pgrep -f "org.apache.catalina.startup.Bootstrap" > /dev/null; then
                        echo "Tomcat is still running."
                        echo "Stopping remaining Tomcat process..."

                        pkill -f "org.apache.catalina.startup.Bootstrap" || true

                        sleep 3
                    fi

                    "$TOMCAT_HOME/bin/startup.sh"

                    echo "Waiting for Tomcat..."
                    sleep 10

                    echo "Tomcat process:"
                    pgrep -af "org.apache.catalina.startup.Bootstrap" || true
                '''
            }
        }

        stage('Verify Application') {
            steps {
                sh '''
                    echo "========================================"
                    echo "Verifying Application"
                    echo "========================================"

                    if curl -f --max-time 15 \
                        http://localhost:8080/LoginWebApp/ \
                        > /tmp/app_response.html
                    then
                        echo "Application is UP."

                        echo "Application response:"
                        head -20 /tmp/app_response.html
                    else
                        echo "Application verification failed."
                        exit 1
                    fi
                '''
            }
        }

        stage('Verify RDS Data') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'rds-db-credentials',
                        usernameVariable: 'RDS_USER',
                        passwordVariable: 'DB_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "========================================"
                        echo "Verifying RDS Database"
                        echo "========================================"

                        mariadb \
                            --skip-ssl-verify-server-cert \
                            -h "$RDS_HOST" \
                            -u "$RDS_USER" \
                            -p"$DB_PASSWORD" \
                            -e "
                                USE $DB_NAME;

                                SHOW TABLES;

                                SELECT COUNT(*) AS USER_COUNT
                                FROM USER;
                            "

                        echo "RDS verification completed successfully."
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
WAR built successfully.
RDS database configured.
WAR deployed to Tomcat.
Application verified.
RDS database verified.
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
