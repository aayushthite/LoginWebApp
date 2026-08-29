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
                                    sh "mvn clean package"
                                    sh "ls -lh target/"
                                    sh "pwd"
                            }
                }

                stage('archive-WAR') {
                        
                            steps {
                                    stash name: 'loginwebapp-war-Aayush',
                                    includes: 'target/*.war'
                                    sh "ls -lh target/"
                                    sh "pwd"
                                }
                 }


                stage('deploy'){
                        
                            agent{
                                    label 'slave-1'
                            }

                            steps{
                                   unstash 'loginwebapp-war-Aayush'
                                   sh "ls -lh target/"
                                   sh "pwd"
                                   sh "sudo cp target/*.war /mnt/apache-tomcat-10.1.59/webapps/"
                            }
                }

        }
}
