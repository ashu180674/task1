
![Screenshot from 2022-12-27 17-41-04](https://user-images.githubusercontent.com/105533911/209665145-61a0d69a-8bc7-4097-be68-64c705162549.png)




In this artitecture i use 2 ubuntu instances one for jenkins and other is to build,upload and deploy docker image.

Image url -->  https://hub.docker.com/repository/docker/ashu180674/project1

I deployd thus app with pipeline as a code the code is -->



node{
    
  stage('git checkout'){
      
      git branch: 'main', url: 'https://github.com/ashu180674/task1.git'
  }
  stage('copy dockerfile to docker server'){
      
      
      sshagent(['slave1']) {
     sh 'ssh -o StrictHostKeyChecking=no ubuntu@43.204.214.101'
     sh 'scp /var/lib/jenkins/workspace/project/* ubuntu@43.204.214.101:/home/ubuntu'
     
      }
      
                 }
  
  stage('build and upload image'){
      
      sshagent(['slave1']) {
     sh 'ssh -o StrictHostKeyChecking=no ubuntu@43.204.214.101 cd /home/ubuntu'
     sh 'ssh -o StrictHostKeyChecking=no ubuntu@43.204.214.101 sudo docker build -t ashu180674/project1:$BUILD_NUMBER .'
     }

  }
  stage('push image to dockerhub'){
      
        sshagent(['slave1']) {
            
        withCredentials([string(credentialsId: 'doc_pass', variable: 'docker_passwd')]) {
         sh 'ssh -o StrictHostKeyChecking=no ubuntu@43.204.214.101 sudo docker login -u ashu180674 -p $docker_passwd'
         sh 'ssh -o StrictHostKeyChecking=no ubuntu@43.204.214.101 sudo docker push ashu180674/project1:$BUILD_NUMBER'
}   
        }  
            
     
     }
  
  stage('deploy web app in dev env'){

 sshagent(['slave1']) {
sh 'ssh -o StrictHostKeyChecking=no ubuntu@43.204.214.101 sudo docker rm -f myapp'
sh 'ssh -o StrictHostKeyChecking=no ubuntu@43.204.214.101 sudo docker run -d -p 80:80 --name myapp ashu180674/project1:${BUILD_NUMBER}'

     }
}

}
