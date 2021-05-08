pipeline {
  agent {
    label 'Linux_slave'
  }
  environment {
    NEXUS_HOST     = '10.10.20.31:8081'
    NEXUS_JAR_REPO = 'storefront_jars_depot'
    NEXUS_PATH     = 'http://${env.NEXUS_ADDRESS}/repository/${env.NEXUS_JAR_REPO}/'
    NEXUS_CRED     = credentials('13f41f0b-b263-4a43-855b-82ffcb0611c0') // => NEXUS_CRED_USR NEXUS_CRED_PSW
    DOCKERHUB_CRED = credentials('aef12c4d-d115-47c1-ad7b-3d8744dc29fa') // => DOCKERHUB_CRED_USR DOCKERHUB_CRED_PSW
    
  }
  stages {
    stage('===> Tests unitaires') {
      steps {
        sh 'mvn test'
      }
     }
    stage('===> Compilation') {
      steps {
        sh 'mvn -B -DskipTests clean package'
      }
    }
    stage ('===> Publication du binaire') {
      steps {
        echo "Current working directory : "
        sh 'pwd'
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file productcatalogue/target/productcatalogue*.jar '${env.NEXUS_PATH}/productcatalogue-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file shopfront/target/*.jar '${env.NEXUS_PATH}/shopfront-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file stockmanager/target/*.jar '${env.NEXUS_PATH}/stockmanager-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
      }
    }
    stage('===> Création des images docker') {
      stages {
        stage('-----> Compilation des images') {
          steps {
            sh "docker build -t frankielemalhonnete/storeapp/productcatalogue:${env.BUILD_NUMBER} ./productcatalogue/"
            sh "docker tag frankielemalhonnete/storeapp/productcatalogue:${env.BUILD_NUMBER}"
            sh "docker tag frankielemalhonnete/storeapp/productcatalogue:latest"
            //-------------------------------------------------------------------------------------------------------
            sh "docker build -t frankielemalhonnete/storeapp/shopfront:${env.BUILD_NUMBER} ./shopfront/"
            sh "docker tag frankielemalhonnete/storeapp/shopfront:${env.BUILD_NUMBER}"
            sh "docker tag frankielemalhonnete/storeapp/shopfront:latest"
            //-------------------------------------------------------------------------------------------------------
            sh "docker build -t frankielemalhonnete/storeapp/stockmanager:${env.BUILD_NUMBER} ./stockmanager/"
            sh "docker tag frankielemalhonnete/storeapp/stockmanager:${env.BUILD_NUMBER}"
            sh "docker tag frankielemalhonnete/storeapp/stockmanager:latest"
          }
        }
        stage('-----> Stockage des images') {
          steps {
            sh "docker login -u ${env.DOCKERHUB_CRED_USR} -p ${env.DOCKERHUB_CRED_PSW}"
            sh "docker push frankielemalhonnete/storeapp/productcatalogue:${env.BUILD_NUMBER}"
            sh "docker push frankielemalhonnete/storeapp/productcatalogue:latest"
            //-------------------------------------------------------------------------------
            sh "docker push frankielemalhonnete/storeapp/shopfront:${env.BUILD_NUMBER}"
            sh "docker push frankielemalhonnete/storeapp/shopfront:latest"
            //-------------------------------------------------------------------------------
            sh "docker push frankielemalhonnete/storeapp/stockmanager:${env.BUILD_NUMBER}"
            sh "docker push frankielemalhonnete/storeapp/stockmanager:latest"
          }
        }
      }
    }
  }
}
