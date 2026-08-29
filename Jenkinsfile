pipeline{

        agent {
                label 'slave-1'
        }

        tools{

                maven 'maven-auto'
        }

        stages{

                stage('complie'){
                        steps{
                                sh "mvn clean package"
                        }
                }

                stage('deploy'){
                        steps{
                                sh "sudo cp /mnt/slave-1/workspace/assignment_11_Project/target/*.war \
                                /mnt/apache-tomcat-10.1.59/webapps/"
                        }
                }
        }
}
