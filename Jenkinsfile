pipeline {
  agent any

  environment {
    DEPLOY_DIR = "/opt/python-webapp"
    SERVICE    = "python-webapp"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build (install deps)') {
      steps {
        sh '''
          python3 -m venv venv
          . venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          . venv/bin/activate
          pytest -q
        '''
      }
    }

    stage('Deploy (localhost)') {
      steps {
        sh """
          set -e

          echo "Deploying to ${DEPLOY_DIR}..."
          sudo mkdir -p ${DEPLOY_DIR}

          # Copy code to deploy dir (exclude venv and git metadata)
          sudo rsync -av --delete --exclude 'venv' --exclude '.git' ./ ${DEPLOY_DIR}/

          echo "Installing dependencies in deploy location..."
          cd ${DEPLOY_DIR}
          python3 -m venv venv
          . venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt

          echo "Restarting service..."
          sudo systemctl restart ${SERVICE}
        """
      }
    }

    stage('Smoke Test') {
      steps {
        sh '''
          echo "Checking app health..."
          curl -f http://localhost:5000/health
        '''
      }
    }
  }

  post {
    always {
      cleanWs()
    }
  }
}
