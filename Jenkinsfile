pipeline {
  agent any

  tools {
    sonarQubeScanner 'SonarScanner'
  }

  environment {
    SONAR_HOST_URL = "http://3.128.205.112:9000"
    SONAR_TOKEN = credentials('Sonarq')
    PYTHON_SERVER_URL = "http://3.128.205.112:5000/upload"
    NVD_API_KEY = credentials('APiNIST')
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/DiegoOC111/Pipeline.git', branch: 'main'
      }
    }

    stage('Install deps') {
      steps {
        sh '''
          python3 -m venv .venv || true
          . .venv/bin/activate

          pip install --upgrade pip

          pip install "Flask==2.2.5" \
                      "Werkzeug==2.2.3" \
                      "Jinja2==3.1.2" \
                      "itsdangerous==2.1.2" \
                      "click==8.1.3"
        '''
      }
    }

    stage('SonarQube Analysis') {
      steps {
        withSonarQubeEnv('SonarServer') {
          sh '''
            sonar-scanner \
              -Dsonar.projectKey=myproject \
              -Dsonar.sources=. \
              -Dsonar.host.url=${SONAR_HOST_URL} \
              -Dsonar.login=${SONAR_TOKEN}
          '''
        }
      }
    }

    stage('Dependency-Check') {
      steps {
        sh '''
          docker run --rm -v $(pwd):/src -v $HOME/.local/share/dependency-check:/report \
            owasp/dependency-check:latest \
            --project myproject --scan /src --format ALL --out /report

          mkdir -p reports
          cp $HOME/.local/share/dependency-check/* reports/ || true
        '''
      }
    }

    stage('Upload Reports to Python Server') {
      steps {
        sh '''
          for f in reports/*; do
            [ -f "$f" ] || continue
            curl -X POST -F "file=@$f" ${PYTHON_SERVER_URL}
          done
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'reports/**', fingerprint: true
    }
    success {
      echo "Build success"
    }
    failure {
      echo "Build failed"
    }
  }
}
