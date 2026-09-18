pipeline{

        agent { 
                label{

                      label 'built-in'
                      customWorkspace '/mnt/project/'
                }
        }


        tools{
              maven 'maven_auto'
        }

        stages{
              stage('package-war-file'){

                      steps{
                              sh 'sudo rm -rf /mnt/project/target/LoginWebApp.war'
                              sh 'mvn clean package'
                      }

              }

        }

}
