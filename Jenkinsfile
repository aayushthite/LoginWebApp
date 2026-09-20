pipeline{

        agent { 
                label{

                      label 'built-in'
                      customWorkspace '/mnt/project/'
                }
        }


        tools{
              maven 'Maven_auto'
        }

        stages{
              stage('package-war-file'){

                      steps{
                              sh 'sudo rm -rf /mnt/web-server/apache-tomcat-10.1.60/webapps/LoginWebApp.war' || true
                              sh 'sudo rm -rf /mnt/project/target/LoginWebApp.war' || true
                              sh 'mvn clean package'
                              sh 'sudo cp /mnt/project/target/LoginWebApp.war /mnt/web-server/apache-tomcat-10.1.60/webapps/'
                              
                      }

              }

        }

}
