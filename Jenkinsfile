pipeline {
  agent any

  stages {
    stage('Init') {
      steps {
        sh 'echo "init"'
      }
    }
    stage('Build') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
          ls -la
          node --version
          npm --version
          npm ci
          npm run build
          echo "---------------"
          ls -la
          echo "Build end"
        '''
      }
    }

    stage('Test') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
          echo "Test stage"
          test -f build/index.html
          echo "Starting app tests"
          npm test
        '''
      }
    }
  }

  post {
    always {
      junit 'test-results/junit.xml'
    }
  }
}