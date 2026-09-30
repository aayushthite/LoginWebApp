pipeline {

    agent any

    tools {
        maven 'Maven_auto'
    }

    environment {
        PROJECT_DIR = '/mnt/project'
        TOMCAT_HOME = '/mnt/web-server/apache-tomcat-10.1.60'

        RDS_HOST = 'velocity-db.c502c4e2yh9.ap-south-1.rds.amazonaws.com'
        DB_NAME  = 'test'
    }

    stages {

        stage('Configure Application Database') {
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
                        echo "Configuring Application Database"
                        echo "========================================"

                        echo "RDS Host:"
                        echo "$RDS_HOST"

                        echo "RDS User:"
                        echo "$RDS_USER"

                        echo "Updating userRegistration.jsp..."

                        sed -i "s|jdbc:mysql://localhost:3306/test|jdbc:mysql://${RDS_HOST}:3306/${DB_NAME}|g" \
                            src/main/webapp/userRegistration.jsp

                        sed -i 's|"root", "root"|"'"${RDS_USER}"'", "'"${DB_PASSWORD}"'"|g' \
                            src/main/webapp/userRegistration.jsp

                        echo "Database configuration updated."

                        echo "Checking configured JDBC URL:"
                        grep -n "DriverManager.getConnection" \
                            src/main/webapp/userRegistration.jsp

                        echo "========================================"
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

                    echo "========================================"
                    echo "WAR created:"
                    echo "========================================"

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

                        echo "========================================"
                        echo "RDS database configuration completed."
                        echo "========================================"
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

                    echo "========================================"
                    echo "Tomcat restart completed."
                    echo "========================================"
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

                        echo "Checking Tomcat logs..."

                        tail -50 "$TOMCAT_HOME/logs/catalina.out" || true

                        exit 1
                    fi

                    echo "========================================"
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

                        echo "========================================"
                        echo "RDS verification completed."
                        echo "========================================"
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

Git checkout                  : SUCCESS
Application DB configuration  : SUCCESS
Maven WAR build               : SUCCESS
RDS database configuration    : SUCCESS
WAR deployment                : SUCCESS
Tomcat restart                : SUCCESS
Application verification      : SUCCESS
RDS verification              : SUCCESS

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

