pipeline {
  agent any

  environment {
    INVENTORY = 'inventory.ini'
  }

  stages {
    stage('Terraform Init & Apply') {
      steps {
        dir('terraform') {
          sh 'terraform init'
          sh 'terraform apply -auto-approve'
        }
      }
    }

    stage('Generate Inventory File') {
      steps {
        script {
          def ip = sh(script: "cd terraform && terraform output -raw public_ip", returnStdout: true).trim()
          writeFile file: "${env.INVENTORY}", text: """
[web]
${ip} ansible_user=azureuser ansible_ssh_private_key_file=~/.ssh/id_rsa
"""
        }
      }
    }

    stage('Run Ansible Playbook') {
      steps {
        sshagent (credentials: ['ansible-key']) {
          sh "ansible-playbook -i ${env.INVENTORY} ansible/install_web.yml"
        }
      }
    }

    stage('Verify Site') {
      steps {
        script {
          def ip = sh(script: "cd terraform && terraform output -raw public_ip", returnStdout: true).trim()
          sh "curl http://${ip}"
        }
      }
    }
  }
}

