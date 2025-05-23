// Jenkins Pipeline to deploy Salesforce metadata across Dev → INT → Stage
pipeline {
  agent any
  stages {
    stage('Test Shell') {
      steps {
        sh 'echo Hello from Jenkins shell'
      }
    }
  }

  environment {
    // Inject Salesforce org auth URLs as secret text credentials from Jenkins
    SF_AUTH_DEV   = credentials('sf-auth-dev')
    SF_AUTH_INT   = credentials('sf-auth-int')
    SF_AUTH_STAGE = credentials('sf-auth-stage')
  }

  stages {

    // 1. Authenticate and deploy to Dev Org
    stage('Authenticate Dev Org') {
      steps {
        // Write the SFDX URL to a file and use sf CLI to login
        writeFile file: 'auth-dev.txt', text: "${SF_AUTH_DEV}"
        sh 'sf org login sfdx-url --sfdx-url-file auth-dev.txt --alias DevOrg --set-default'
      }
    }

    stage('Deploy to Dev') {
      steps {
        // Deploy metadata to Dev org
        sh 'sf deploy metadata --target-org DevOrg --source-dir force-app --test-level RunLocalTests'
      }
    }

    // 2. Authenticate and deploy to INT Org
    stage('Authenticate INT Org') {
      steps {
        writeFile file: 'auth-int.txt', text: "${SF_AUTH_INT}"
        sh 'sf org login sfdx-url --sfdx-url-file auth-int.txt --alias IntOrg'
      }
    }

    stage('Deploy to INT') {
      steps {
        sh 'sf deploy metadata --target-org IntOrg --source-dir force-app --test-level RunLocalTests'
      }
    }

    // 3. Manual QA approval before Stage deployment
    stage('QA Approval for Stage') {
      steps {
        input message: 'Has QA approved the deployment to Stage?'
      }
    }

    // 4. Authenticate and deploy to Stage Org
    stage('Authenticate Stage Org') {
      steps {
        writeFile file: 'auth-stage.txt', text: "${SF_AUTH_STAGE}"
        sh 'sf org login sfdx-url --sfdx-url-file auth-stage.txt --alias StageOrg'
      }
    }

    stage('Deploy to Stage') {
      steps {
        sh 'sf deploy metadata --target-org StageOrg --source-dir force-app --test-level RunLocalTests'
      }
    }
  }
}