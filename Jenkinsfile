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
        
              stage('package-war_FILE'){

                      steps{
                              sh 'mvn clean package'
                      }

              }

              stage('Ansible--check'){
              
                        steps{
                                sh 'ansible-playbook -i hosts test.yaml --syntax-check'
                        
                        }
              }

              
              stage('Deploy'){
              
                        steps{
                                sh 'ansible-playbook -i hosts test.yaml'
                        
                        }
              }
        
        
      }

}
