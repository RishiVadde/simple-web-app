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

    stage('Package Artifact (tar.gz)') {
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
        sh """
          set -e

          echo "Preparing deploy folder: ${DEPLOY_DIR}"
          mkdir -p ${DEPLOY_DIR}

          echo "Cleaning old app files (keeping logs/pid if present)..."
          rm -rf ${DEPLOY_DIR}/* || true

          echo "Extracting artifact..."
          tar -xzf ${ARTIFACT_TAR} -C ${DEPLOY_DIR}

          echo "Creating venv + installing dependencies in deploy folder..."
          cd ${DEPLOY_DIR}
          python3 -m venv venv
          . venv/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt

          echo "Restarting app (no systemd, no sudo)..."

          # Stop old process (PID file)
          if [ -f "${PID_FILE}" ]; then
            OLD_PID=\$(cat "${PID_FILE}" || true)
            if [ -n "\${OLD_PID}" ] && ps -p "\${OLD_PID}" > /dev/null 2>&1; then
              echo "Stopping old PID: \${OLD_PID}"
              kill "\${OLD_PID}" || true
              sleep 2
            fi
          fi

          # Optional safety: stop any gunicorn running app:app
          pkill -f "gunicorn.*app:app" || true

          # Start new gunicorn
          nohup ${DEPLOY_DIR}/venv/bin/gunicorn -b 0.0.0.0:${PORT} app:app > ${LOG_FILE} 2>&1 &
          echo \$! > ${PID_FILE}

          echo "Started new PID: \$(cat ${PID_FILE})"
          sleep 2
        """
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
