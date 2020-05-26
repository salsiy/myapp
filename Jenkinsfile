//jenkinsfile
pipeline{
        agent { node { label 'docker' }}
        stages{
                stage(' stage 1'){
                        steps{
                                echo 'stage1'
                        }
                }
                stage(' stage 3'){
                        steps{
                                script{
                                
                                        withDockerRegistry(credentialsId: 'gitlab-user', url: 'https://user.siyad.com:5050')
                                        {
                                                def devImage = docker.build('user.siyad.com:5050/test/test-app/alpha1:latest')
                                                devImage.push()
                                        }
                                }
                        }

                
                }
                stage('Deployment'){
                        steps{
                               ansiblePlaybook(
                    inventory: 'inventory.yml',
                    playbook: 'playbook.yml' ,
		    credentialsId: 'ansible-user',
                    disableHostKeyChecking: true,
                    colorized : 'true',
	            extraVars : [resetenv: 'dev'],
                    vaultCredentialsId : 'ansible-vault'

                              )
                        }
                }
        }
        post {
                always{
                         echo 'Notify GitLab'
                         updateGitlabCommitStatus name: 'build', state: 'pending'
                         updateGitlabCommitStatus name: 'build', state: 'success'
			 //cleanWs()
                        
                }
                failure{ //Send an email to the person that broke the build
			updateGitlabCommitStatus name: 'build', state: 'failed'
                        step([$class            : 'Mailer',
                                notifyEveryUnstableBuild: true,
                                recipients              : [emailextrecipients([[$class: 'CulpritsRecipientProvider'], [$class: 'RequesterRecipientProvider']])].join(' ')])
                }
        }
}
