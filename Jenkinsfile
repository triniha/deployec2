pipeline {
  agent any
  
  environment {
    AWS_ACCESS_KEY_ID = credentials
    EC2_INSTANCE_TYPE = 't2.micro'
    GITHUB_REPO = 'https://github.com/surajn17/deploy.git'
    GITHUB_BRANCH = 'main'
  }
  
  stages {
    stage('Launch EC2 Instance') {
      steps {
        script {
          def instanceId = awsEc2LaunchInstance( 
            ami: '',
            instanceType: env.EC2_INSTANCE_TYPE,
            region: env.AWS_REGION,
            securityGroupIds: 'your-security-group-id',
            subnetId: 'your-subnet-id',
            keyPair: 'your-key-pair-name'
          )
          echo "Launched EC2 instance with ID: $instanceId"
          awsEc2AssociatePublicIpAddress(instanceId: instanceId, region: env.AWS_REGION)
          awsEc2WaitInstanceStatus(instanceIds: instanceId, region: env.AWS_REGION, desiredStatus: 'running')
          def instanceIpAddress = awsEc2DescribeInstances(instanceIds: instanceId, region: env.AWS_REGION)[0].publicIpAddress
          echo "Instance IP Address: $instanceIpAddress"
          
          // Store the instance IP address for later stages
          env.INSTANCE_IP_ADDRESS = instanceIpAddress
        }
      }
    } 
stage('Check Dependencies') {
      steps {
        script {
          checkDependencies()
        }
      }
    

    stage('Create WordPress Site') {
      when {
        expression {
          params.action == 'create' && params.siteName
        }
      }
      steps {
        script {
          createWordPressSite(params.siteName)
          addHostsEntry(params.siteName)
          openInBrowser(params.siteName)
        }
      }
    }

    stage('Enable Site') {
      when {
        expression {
          params.action == 'enable'
        }
      }
      steps {
        script {
          enableSite()
        }
      }
    }

    stage('Disable Site') {
      when {
        expression {
          params.action == 'disable'
        }
      }
      steps {
        script {
          disableSite()
        }
      }
    }

    stage('Delete Site') {
      when {
        expression {
          params.action == 'delete' && params.siteName
        }
      }
      steps {
        script {
          deleteSite(params.siteName)
        }
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}

def checkDependencies() {
  sh "if ! command -v docker &> /dev/null; then echo 'Docker is not installed. Installing...'; installDocker; fi"
  sh "if ! command -v docker-compose &> /dev/null; then echo 'Docker Compose is not installed. Installing...'; installDockerCompose; fi"
}

def installDocker() {
  sh """
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl software-properties-common
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    sudo apt update
    sudo apt install -y docker-ce docker-ce-cli containerd.io
    sudo usermod -aG docker "\$USER"
  """
}

def installDockerCompose() {
  sh """
    sudo curl -fsSL -o /usr/local/bin/docker-compose https://github.com/docker/compose/releases/latest/download/docker-compose-\$(uname -s)-\$(uname -m)
    sudo chmod +x /usr/local/bin/docker-compose
  """
}

def createWordPressSite(siteName) {
  sh """
    mkdir "\$siteName"
    cd "\$siteName"
    cat << EOF > docker-compose.yml
version: '3'
services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    volumes:
      - ./wp-content:/var/www/html/wp-content
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data:
EOF
    docker-compose up -d
  """
}

def addHostsEntry(siteName) {
  sh """
    echo "127.0.0.1 \$siteName" | sudo tee -a /etc/hosts > /dev/null
  """
}

def openInBrowser(siteName) {
  echo "Open http://\$siteName in a browser."
}

def enableSite() {
  sh "docker-compose start"
}

def disableSite() {
  sh "docker-compose stop"
}

def deleteSite(siteName) {
  sh """
    cd "\$siteName"
    docker-compose down
    cd ..
    rm -rf "\$siteName"
  """
}
