pipeline {
  agent any

  stages {
    stage('Launch EC2 Instance') {
      steps {
        sh '''
          aws ec2 run-instances \
            --image-id ami-0d52744d6551d851e \
            --instance-type t2.micro \
            --key-name tokyo \
            --security-group-ids sg-0f9f4e460cf0f1fa1 \
            --subnet-id subnet-032c52c82cabfb6e0 \
            --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=jenkinsmadeec2}]'
        '''
      }
    }
    stage('git clone') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@54.238.215.227 git clone https://github.com/surajn17/docker-com.git'
          }
      }
    }
    stage('change directory') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@54.238.215.227 cd docker-com'
          }
      }
    }
    stage('Installation of docker and docker-compose') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@54.238.215.227 "cd docker-com; sudo chmod a+x docker.sh; sudo ./docker.sh" '
          }
      }
    }
    stage('deploy wordpress') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@54.238.215.227 "cd docker-com; sudo docker-compose up -d" '
          }
      }
    }
  }
}
