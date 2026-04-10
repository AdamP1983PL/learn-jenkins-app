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

    stage('Deploy staging') {
      agent {
        docker {
          image 'node:18-alpine'
          reuseNode true
        }
      }
      steps {
        sh '''
          npm install netlify-cli@20.1.1
          npm install node-jq
          node_modules/.bin/netlify --version
          echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
          node_modules/.bin/netlify status
          node_modules/.bin/netlify deploy --dir=build  --json > deploy-output.json
          node_modules/.bin/node-jq -r '.deploy_url' deploy-output.json
        '''
      }
    }

//     stage('Approval') {
//       steps {
//         timeout(time:2, unit: 'MINUTES') {
//           input message: 'Ready to deploy?'
//         }
//       }
//     }

//     stage('Deploy prod') {
//       agent {
//         docker {
//           image 'node:18-alpine'
//           reuseNode true
//         }
//       }
//       steps {
//         sh '''
//           npm install netlify-cli@20.1.1
//           node_modules/.bin/netlify --version
//           echo "Deploying to production. Site ID: $NETLIFY_SITE_ID"
//           node_modules/.bin/netlify status
//           node_modules/.bin/netlify deploy --dir=build --prod
//         '''
//       }
//     }
//
//     stage('Prod E2E') {
//       agent {
//         docker {
//           image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
//           reuseNode true
//         }
//       }
//
//       environment {
//         CI_ENVIRONMENT_URL = 'https://quiet-pegasus-7b0a58.netlify.app'
//       }
//
//       steps {
//         sh '''
//           echo "Starting Prod E2E tests"
//           npx playwright test --reporter=html
//         '''
//       }
//     }

  }

  post {
    always {
      junit 'jest-results/junit.xml'
    }
  }
}