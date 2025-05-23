// Jenkins Pipeline to deploy Salesforce metadata to Dev Org
pipeline {
  agent any  // Use any available Jenkins agent

  environment {
    // Inject the secret text credential named 'sf-auth-dev' into the pipeline
    SF_AUTH_DEV = credentials('sf-auth-dev')
  }

  stages {
    stage('Install Salesforce CLI') {
      steps {
        // Install Salesforce CLI globally
        sh 'npm install sfdx-cli --global'
      }
    }

    stage('Authenticate to Dev Org') {
      steps {
        // Write the Auth URL secret into a file
        writeFile file: 'auth.txt', text: "${SF_AUTH_DEV}"
        
        // Authenticate to the Dev Org using the Auth URL
        // -a DevOrg sets the alias name to 'DevOrg'
        sh 'sfdx auth:sfdxurl:store -f auth.txt -a DevOrg'
      }
    }

    stage('Deploy to Dev Org') {
      steps {
        // Deploy metadata from the force-app directory to DevOrg
        // RunLocalTests ensures Apex tests in the org are executed
        sh 'sfdx force:source:deploy -u DevOrg -p force-app --testlevel RunLocalTests'
      }
    }
  }
}
