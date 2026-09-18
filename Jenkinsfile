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
                              sh 'mvn clean package'
                      }

              }

        }

}
