pipeline{
        
        agent{
                label 'built-in'
        }

        tools{
                maven 'maven-auto'
        }            
        
        stages{
                stage('complie'){

                            steps{
                                    sh "rm -rf /root/.m2/repository"
                                    sh "rm -rf /root/.jenkins/workspace/LoginWebApp/target/*"
                                    sh "mvn clean install"
                            }
                }

                stage('deploy'){
                        
                            agent{
                                    label 'slave-1'
                            }

                            steps{
                                    echo "Hello"
                            }
                }

        }
}
