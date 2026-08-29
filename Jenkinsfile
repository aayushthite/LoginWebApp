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
                                sh "sudo cp /targets/*.war /mnt/apache-tomcat-10.1.59/webapps/"
                        }
                }
        }
}
