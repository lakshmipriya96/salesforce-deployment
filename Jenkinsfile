// This Jenkinsfile automates deployments to Dev, INT, and Stage Salesforce orgs.
// It uses SFDX CLI, credentials stored in Jenkins, and includes a manual QA approval step before Stage.

pipeline {
  agent any  // Run on any available Jenkins agent
  docker {
      image 'node:18' // Node.js with npm
    }

  environment {
    // These are secret text credentials stored in Jenkins → Credentials → Global
    SF_AUTH_DEV   = credentials('sf-auth-dev')
    SF_AUTH_INT   = credentials('sf-auth-int')
    SF_AUTH_STAGE = credentials('sf-auth-stage')
  }

  stages {
    // 1. Install Salesforce CLI (SFDX)
    stage('Install SFDX CLI') {
      steps {
        sh 'npm install sfdx-cli --global'
      }
    }

    // 2. Authenticate to Dev Org
    stage('Authenticate Dev Org') {
      steps {
        // Write the Dev auth URL to a file, then auth using SFDX
        writeFile file: 'auth-dev.txt', text: "${SF_AUTH_DEV}"
        sh 'sfdx auth:sfdxurl:store -f auth-dev.txt -a DevOrg'
      }
    }

    // 3. Deploy to Dev Org
    stage('Deploy to Dev') {
      steps {
        sh 'sfdx force:source:deploy -u DevOrg -p force-app --testlevel RunLocalTests'
      }
    }

    // 4. Authenticate to INT Org
    stage('Authenticate INT Org') {
      steps {
        writeFile file: 'auth-int.txt', text: "${SF_AUTH_INT}"
        sh 'sfdx auth:sfdxurl:store -f auth-int.txt -a IntOrg'
      }
    }

    // 5. Deploy to INT Org
    stage('Deploy to INT') {
      steps {
        sh 'sfdx force:source:deploy -u IntOrg -p force-app --testlevel RunLocalTests'
      }
    }

    // 6. Pause and wait for QA to approve deployment to Stage
    stage('QA Approval for Stage') {
      steps {
        input message: 'Has QA approved deployment to Stage?'
      }
    }

    // 7. Authenticate to Stage Org
    stage('Authenticate Stage Org') {
      steps {
        writeFile file: 'auth-stage.txt', text: "${SF_AUTH_STAGE}"
        sh 'sfdx auth:sfdxurl:store -f auth-stage.txt -a StageOrg'
      }
    }

    // 8. Deploy to Stage Org
    stage('Deploy to Stage') {
      steps {
        sh 'sfdx force:source:deploy -u StageOrg -p force-app --testlevel RunLocalTests'
      }
    }
  }
}
