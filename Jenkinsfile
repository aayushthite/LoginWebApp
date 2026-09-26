pipeline {

    agent {
        label {
            label 'built-in'
            customWorkspace '/mnt/project/'
        }
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
                    sudo rm -rf /root/temp/apache-tomcat-10.1.60/webapps/LoginWebApp
                    sudo rm -f /root/temp/apache-tomcat-10.1.60/webapps/LoginWebApp.war

                    sudo cp /mnt/project/target/LoginWebApp.war \
                    /root/temp/apache-tomcat-10.1.60/webapps
                '''
            }
        }

        stage('Restart Tomcat') {
            steps {
                sh '''
                    sudo /root/temp/apache-tomcat-10.1.60/bin/shutdown.sh || true
                    sleep 10
                    sudo /root/temp/apache-tomcat-10.1.60//bin/startup.sh
                '''
            }
        }

    }
}
