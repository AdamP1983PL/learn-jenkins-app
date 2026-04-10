pipeline {
  agent any

  environment {
    NETLIFY_SITE_ID = 'b392ce51-5e64-4e4c-a1ea-ec2e35547015'
    NETLIFY_AUTH_TOKEN = credentials('netlify-token')
  }

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
          node --version
          npm --version
          npm ci
          npm run build
          echo "Build end"
          echo "Build end"
        '''
      }
    }

    stage('Run Test in parallel') {
      parallel {
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

//         stage('E2E') {
//           agent {
//             docker {
//               image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
//               reuseNode true
//             }
//           }
//           steps {
//             sh '''
//               echo "Starting E2E tests"
//               npm install serve
//               node_modules/.bin/serve -s build &
//               sleep 10
//               npx playwright test --reporter=html
//             '''
//           }
//         }
      }
    }

    stage('Deploy') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
          npm install netlify-cli@20.1.1
          node_modules/.bin/netlify --version
          echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
          node_modules/.bin/netlify status
          node_modules/.bin/netlify deploy --dir=build --prod
        '''
      }
    }

  }

  post {
    always {
      junit 'jest-results/junit.xml'
    }
  }
}