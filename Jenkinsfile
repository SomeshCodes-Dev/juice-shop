pipeline {
  agent any

  environment {
    PATH = "/opt/homebrew/bin:${env.PATH}"
  }

  options {
    skipDefaultCheckout(false)
    timestamps()
  }

  stages {

    stage('Environment Check') {
      steps {
        sh '''
          echo "Checking Node and npm..."
          which node
          which npm
          node --version
          npm --version
        '''
      }
    }

    stage('Verify Repository') {
      steps {
        sh '''
          echo "Jenkins workspace:"
          pwd

          echo "Juice Shop files:"
          ls -la

          echo "npm registry:"
          npm config get registry
        '''
      }
    }

    stage('Install Dependencies') {
      steps {
    sh 'npm install --legacy-peer-deps'
      }
    }

    stage('Build Juice Shop') {
      steps {
        sh 'npm run build'
      }
    }

    stage('Nexus IQ Scan') {
      steps {
        script {
          def policyEvaluation = nexusPolicyEvaluation(
            failBuildOnNetworkError: true,
            iqApplication: selectedApplication('juice-shop'),
            iqScanPatterns: [
              [scanPattern: 'package.json'],
              [scanPattern: 'package-lock.json']
            ],
            iqStage: 'build',
            jobCredentialsId: 'iq-admin'
          )

          echo "IQ Report: ${policyEvaluation.applicationCompositionReportUrl}"
        }
      }
    }
  }

  post {
    always {
      archiveArtifacts(
        artifacts: 'package.json, package-lock.json',
        fingerprint: true,
        allowEmptyArchive: true
      )
    }
  }
}
