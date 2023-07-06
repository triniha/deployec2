pipeline {
  agent any

  stages {
    stage('Install Nginx') {
      steps {
          sshagent(['ssh-agent']) {
           sh 'ssh -tt -o StrictHostKeyChecking=no ubuntu@18.181.181.79 
            // Update these variables accordingly
            def projectName = 'wordpress-site'
            def dockerComposeFile = 'docker-compose.yml'

            // Check if Docker and Docker Compose are installed
            checkDependencies()

            // Create WordPress site
            createWordPressSite(projectName, dockerComposeFile)
            def checkDependencies() {
            // Check if Docker is installed
            if (!sh(returnStatus: true, script: 'command -v docker').toBoolean()) {
            echo "Docker is not installed. Installing Docker..."
            // Install Docker
            sh 'curl -fsSL https://get.docker.com -o get-docker.sh'
            sh 'sudo sh get-docker.sh'
            sh 'sudo usermod -aG docker "$USER"'
            sh 'rm get-docker.sh'
            echo "Docker has been installed."
            } else {
            echo "Docker is already installed."
            }

            // Check if Docker Compose is installed
            if (!sh(returnStatus: true, script: 'command -v docker-compose').toBoolean()) {
            echo "Docker Compose is not installed. Installing Docker Compose..."
            // Install Docker Compose
            sh 'sudo curl -fsSL -o /usr/local/bin/docker-compose https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)'
            sh 'sudo chmod +x /usr/local/bin/docker-compose'
            echo "Docker Compose has been installed."
            } else {
            echo "Docker Compose is already installed."
            }
            }

            def createWordPressSite(projectName, dockerComposeFile) {
            // Create the site directory
            sh "mkdir -p $projectName"
            sh "cd $projectName"

            // Copy the Docker Compose file
            sh "cp $dockerComposeFile $projectName/"

            // Start the containers
            sh "docker-compose up -d"

            echo "WordPress site has been created. You can access it at http://<SSH_NODE_IP>."
            }'
          }
      }
    }
  }
}
