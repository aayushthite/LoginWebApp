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
                                    sh "mvn clean package"
                                    sh "pwd"
                                    sh "whoami && echo $HOME"
                            }
                }

                stage('stash-WAR') {
                        
                            steps {
                                    sh "pwd"    
                                    stash name: 'LoginWebApp-war',
                                    includes: 'target/*.war'
                                    sh "pwd"
                                    sh "whoami && echo $HOME"
                                    
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
                                   sh "whoami && echo $HOME"
                                   sh "sudo cp target/*.war /mnt/apache-tomcat-10.1.59/webapps/"
                            }
                }

        }
}
