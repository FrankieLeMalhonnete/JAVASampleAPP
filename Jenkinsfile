pipeline {
  agent {
    label 'Linux'
  }
  stages {
    stage ('Script') {
      steps {
        sh 'chmod +x test_mb_jenkins.sh'
        sh './test_mb_jenkins.sh'
      }
    }
  }
}
