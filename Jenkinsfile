pipeline {
  agent any

  options {
    timestamps()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  stages {

    stage('Build') {
      steps {
        sh '''
          rm -rf report
          mkdir -p report

          cat > report/index.html <<'EOF'
<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Hello from Jenkins</title>
  <style>
    body{font-family:Arial;background:#0b1220;color:#fff;display:grid;place-items:center;height:100vh;margin:0}
    .card{background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.15);padding:28px;border-radius:18px;text-align:center;width:min(650px,92%)}
    h1{margin:0 0 10px}
    button{padding:10px 14px;border-radius:12px;border:0;background:linear-gradient(135deg,#5eead4,#60a5fa);font-weight:700;cursor:pointer}
    .btn2{background:rgba(255,255,255,.12);color:#fff;border:1px solid rgba(255,255,255,.2);margin-left:10px}
  </style>
</head>
<body>
  <div class="card">
    <h1>Hello from Jenkins ✅</h1>
    <p>This is a styled HTML report published from Jenkins.</p>
    <button onclick="alert('Build Successful!')">Click Me</button>
    <button class="btn2" onclick="document.body.style.background='#111827'">Change Theme</button>
  </div>
</body>
</html>
EOF
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          # simple test - file exists
          test -f report/index.html
        '''
      }
    }

    stage('Deploy (Publish HTML Artifact)') {
      steps {
        publishHTML(target: [
          reportDir: 'report',
          reportFiles: 'index.html',
          reportName: 'Hello from Jenkins (Styled)',
          keepAll: true,
          alwaysLinkToLastBuild: true,
          allowMissing: false
        ])
      }
    }
  }

  post {
    always { cleanWs() }
  }
}
