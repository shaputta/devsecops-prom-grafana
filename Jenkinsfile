pipeline {
  agent {
    label 'sha-ubuntu-node'
  }
  environment {
    APP_NAME = "flask-counter"
    TARGET_URL = "http://98.80.137.39"
  }
  stages {
    stage('Build Docker Image') {
      steps {
        sh '''
          cd app
          docker rm -f ${APP_NAME} || true
          docker image rmi -f ${APP_NAME}:latest || true
          docker build -t ${APP_NAME}:latest .
        '''
      }
    }
  
    stage('Run App Container') {
      steps {
        sh '''

          docker run -d --name ${APP_NAME} --network prom-grafana_monitoring -p 8010:5000 ${APP_NAME}:latest
        '''
      }
    }
    stage('Validate Metrics') {
      steps {
        sh '''
          echo "Prometheus Targets:"
          curl -s ${TARGET_URL}:8006/api/v1/targets | jq '.data.activeTargets'
        '''
      }
    }
    stage('Health Check') {
      steps {
        script {
          def resp = sh(script: "curl -s -o /dev/null -w '%{http_code}' ${TARGET_URL}:8010", returnStdout: true).trim()
          if (resp != '200') {
            error "Application failed health check!"
          } else {
            echo "App is healthy ✅"
          }
        }
      }
    }
  }
  post {
    always {
      sh 'docker ps'
    }
  }
}