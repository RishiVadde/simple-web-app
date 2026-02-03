pipeline {
  agent any

  environment {
    APP_NAME     = "python-webapp"
    DEPLOY_DIR   = "/var/lib/jenkins/python-webapp"     // writable by Jenkins (no sudo)
    PORT         = "5000"

    // tar artifact name
    ARTIFACT_TAR = "dist/${APP_NAME}-${env.BUILD_NUMBER}.tar.gz"

    PID_FILE     = "${DEPLOY_DIR}/${APP_NAME}.pid"
    LOG_FILE     = "${DEPLOY_DIR}/${APP_NAME}.log"
  }

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
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

    stage('Package Artifact') {
      steps {
        sh """
          rm -rf dist
          mkdir -p dist

          # Create tar.gz artifact excluding venv, git, caches, dist
          tar --exclude='./venv' \
              --exclude='./.git' \
              --exclude='./__pycache__' \
              --exclude='./dist' \
              --exclude='*.pyc' \
              -czf ${ARTIFACT_TAR} .
        """
      }
    }

    stage('Archive Artifact') {
      steps {
        archiveArtifacts artifacts: 'dist/*.tar.gz', fingerprint: true
      }
    }

    stage('Deploy from Artifact (localhost)') {
      
      steps {
        publishHTML(target: [
          reportDir: 'report',
          reportFiles: 'index.html',
          reportName: 'Hello from Jenkins (Styled)',
          keepAll: true,
          alwaysLinkToLastBuild: true
          ])
      }

    }

    stage('Smoke Test') {
      steps {
        sh '''
          echo "Smoke testing..."
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
