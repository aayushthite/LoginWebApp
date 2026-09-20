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

        stage('Build WAR') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Deploy WAR') {
            steps {
                sh '''
                    sudo rm -f /mnt/web-server/apache-tomcat-10.1.60/webapps/LoginWebApp.war

                    sudo cp /mnt/project/target/LoginWebApp.war \
                    /mnt/web-server/apache-tomcat-10.1.60/webapps/
                '''
            }
        }

    }
}
```
