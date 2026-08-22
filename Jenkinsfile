pipeline{

    agent{
            label{
                    label 'built-in'
                    customWorkspace '/mnt/project'
            }
    }

    tools{
        maven 'maven-3.9.16'
    }

    stages{
            stage(package){
                
                    steps{
                        sh "sudo rm -rf /root/.m2/repository/"
                        sh "mvn clean install "
                    }
            }

            stage(deploy){

                    steps{

                        sh "sudo cp /mnt/project/target/*.war /mnt/web-server/apache-tomcat-10.1.59/webapps/"
                    }

            }

    }

}
