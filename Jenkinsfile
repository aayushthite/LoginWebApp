pipeline{
          label {
                  label 'built-in'
                  customWorkspace '/mnt/project/'
          }


      tools{
              maven 'Maven_auto'
      }

      stages{
        
              stage('package'){

                      steps{
                              sh "clean package"
                      }

              }
        
        
      }

}
