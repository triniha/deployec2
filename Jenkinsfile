pipeline {
  agent any

  stages {
    stage('git clone') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@52.199.172.31 git clone https://github.com/surajn17/docker-com.git'
          }
      }
    }
    stage('change directory') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@52.199.172.31 cd docker-com'
          }
      }
    }
    stage('Installation of docker and docker-compose') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@52.199.172.31 "cd docker-com; sudo chmod a+x docker.sh; sudo ./docker.sh" '
          }
      }
    }
    stage('deploy wordpress') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@52.199.172.31 "cd docker-com; sudo docker-compose up -d" '
          }
      }
    }
  }
}
