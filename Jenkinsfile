pipeline{
        
        agent{
                label 'built-in'
        }

        tools{
                maven 'maven-auto'
        }            
        
        stages{
                stage('clone'){
                        steps{
                                git: 'https://github.com/aayushthite/LoginWebApp.git'
                        }
                }

                stage('complie'){
                        steps{
                                sh "mvn clean install"
                        }
                }

                stage('deploy'){
                        
                        label{
                                label 'slave-1'
                        }

                        steps{
                                echo "Hello"
                        }
                }

        }
}
