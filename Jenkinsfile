pipeline {
  agent {
    label 'Linux_slave'
  }
  environment {
    NEXUS_ADRESS   = '10.10.20.31:8081'
    NEXUS_JAR_REPO = 'storefront_jars_depot'
    NEXUS_CRED     = credentials('13f41f0b-b263-4a43-855b-82ffcb0611c0') // => NEXUS_CRED_USR NEXUS_CRED_PSW
    DOCKERHUB_CRED = credentials('aef12c4d-d115-47c1-ad7b-3d8744dc29fa') // => DOCKERHUB_CRED_USR DOCKERHUB_CRED_PSW
  }
  stages {
    stage('Test unitaires') {
      steps {
        sh 'mvn test'
      }
     }
    stage('Compilation') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
    }
    stage ('Publication du binaire') {
      steps {
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file target/*.war 'http://${env.NEXUS_ADDRESS}/repository/${env.NEXUS_JAR_REPO}/app${BUILD_NUMBER}.war'"
      }
    }
  }
}
