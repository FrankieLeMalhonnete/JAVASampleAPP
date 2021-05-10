pipeline {
  agent {
    label 'Linux_slave'
  }
  environment {
    NEXUS_HOST          = 'http://10.10.20.31:8081'
    NEXUS_JAR_REPO_PROD = 'repository/storefront_jars_depot'
    NEXUS_JAR_REPO_DEV  = 'repository/storefront_jars_depot_develop'
    NEXUS_CRED          = credentials('13f41f0b-b263-4a43-855b-82ffcb0611c0') // => NEXUS_CRED_USR NEXUS_CRED_PSW
    DOCKERHUB_CRED      = credentials('aef12c4d-d115-47c1-ad7b-3d8744dc29fa') // => DOCKERHUB_CRED_USR DOCKERHUB_CRED_PSW
  }
  //tools {
  //  maven 'mvn3.6.0'
  //  jdk 'jdk1.8.0'
  //}
  stages {
    stage('===> Tests unitaires') {
      steps {
        echo "Current user : "
        sh 'whoami'
        sh 'mvn test'
      }
    }
    stage('===> Compilation') {
      //parallel {
        //stage ('Maven build') {
          steps {
            sh 'mvn -B -DskipTests clean package'
          }
        //}
        //stage ('Checkstyle') {
        //  steps {
        //    sh "mvn checkstyle:check"
        //    recordIssues(tools: [checkStyle(reportEncoding: 'UTF-8')])
        //  }
        //}
      //}
    }
    stage ('===> Publication du binaire de prod') {
      when {
        branch 'main'
      }
      steps {
        echo "Current working directory : "
        sh 'pwd'
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file productcatalogue/target/productcatalogue*.jar '${env.NEXUS_HOST}/${env.NEXUS_JAR_REPO_PROD}/productcatalogue-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file shopfront/target/*.jar '${env.NEXUS_HOST}/${env.NEXUS_JAR_REPO_PROD}/shopfront-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file stockmanager/target/*.jar '${env.NEXUS_HOST}/${env.NEXUS_JAR_REPO_PROD}/stockmanager-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
      }
    }
    stage ('===> Publication du binaire de branche develop') {
      when {
        branch 'develop'
      }
      steps {
        echo "Current working directory : "
        sh 'pwd'
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file productcatalogue/target/productcatalogue*.jar '${env.NEXUS_HOST}/${env.NEXUS_JAR_REPO_DEV}/productcatalogue-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file shopfront/target/*.jar '${env.NEXUS_HOST}/${env.NEXUS_JAR_REPO_DEV}/shopfront-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
        sh "curl -u ${env.NEXUS_CRED_USR}:${env.NEXUS_CRED_PSW} --upload-file stockmanager/target/*.jar '${env.NEXUS_HOST}/${env.NEXUS_JAR_REPO_DEV}/stockmanager-0.0.1-SNAPSHOT-${BRANCH_NAME}-${BUILD_NUMBER}.jar'"
      }
    }
    stage('===> Création des images docker') {
      stages {
        stage('-----> Compilation des images') {
          steps {
            echo "Current working directory : "
            sh 'pwd'
            echo "Current user : "
            sh 'whoami'
            sh "docker login -u ${env.DOCKERHUB_CRED_USR} -p ${env.DOCKERHUB_CRED_PSW}"
            sh "docker build -t frankielemalhonnete/productcatalogue:${BUILD_NUMBER} productcatalogue/"
            sh "docker tag frankielemalhonnete/productcatalogue:${BUILD_NUMBER} frankielemalhonnete/productcatalogue:${BRANCH_NAME}-${BUILD_NUMBER}"
            sh "docker tag frankielemalhonnete/productcatalogue:${BUILD_NUMBER} frankielemalhonnete/productcatalogue:latest"
            //-------------------------------------------------------------------------------------------------------
            sh "docker build -t frankielemalhonnete/shopfront:${BUILD_NUMBER} shopfront/"
            sh "docker tag frankielemalhonnete/shopfront:${BUILD_NUMBER} frankielemalhonnete/shopfront:${BRANCH_NAME}-${BUILD_NUMBER}"
            sh "docker tag frankielemalhonnete/shopfront:${BUILD_NUMBER} frankielemalhonnete/shopfront:latest"
            //-------------------------------------------------------------------------------------------------------
            sh "docker build -t frankielemalhonnete/stockmanager:${BUILD_NUMBER} stockmanager/"
            sh "docker tag frankielemalhonnete/stockmanager:${BUILD_NUMBER} frankielemalhonnete/stockmanager:${BRANCH_NAME}-${BUILD_NUMBER}"
            sh "docker tag frankielemalhonnete/stockmanager:${BUILD_NUMBER} frankielemalhonnete/stockmanager:latest"
            sh "docker logout"
          }
        }
        stage ('-----> Verification de l\'image productcatalogue') {
          agent {
            docker {
              image 'frankielemalhonnete/productcatalogue:${BUILD_NUMBER}'
              args '-p 9001:9001'
            }
          }
          steps {
            script {
              sh 'curl localhost:9001'
            }
          }
        }
        stage('-----> Verification de l\'image shopfront') {
          agent {
            docker {
              image 'frankielemalhonnete/shopfront:${BUILD_NUMBER}'
              args '-p 9002:9002'
            }
          }
          steps {
            script {
              sh 'curl localhost:9002'
            }
          }
        }
        stage('-----> Verification de l\'image stockmanager') {
          agent {
            docker {
              image 'frankielemalhonnete/stockmanager:${BUILD_NUMBER}'
              args '-p 9003:9003'
            }
          }
          steps {
            script {
              sh 'curl localhost:9003'
            }
          }
        }
        stage('-----> Stockage des images') {
          steps {
            sh "docker login -u ${env.DOCKERHUB_CRED_USR} -p ${env.DOCKERHUB_CRED_PSW} "
            sh "docker push frankielemalhonnete/productcatalogue:${BRANCH_NAME}-${BUILD_NUMBER}"
            sh "docker push frankielemalhonnete/productcatalogue:latest"
            //-------------------------------------------------------------------------------
            sh "docker push frankielemalhonnete/shopfront:${BRANCH_NAME}-${BUILD_NUMBER}"
            sh "docker push frankielemalhonnete/shopfront:latest"
            //-------------------------------------------------------------------------------
            sh "docker push frankielemalhonnete/stockmanager:${BRANCH_NAME}-${BUILD_NUMBER}"
            sh "docker push frankielemalhonnete/stockmanager:latest"
            sh "docker logout"
          }
        }
      }
    }
    stage ('Deploiement sur EKS') {
      steps {
        withKubeConfig({credentialsID: 'Kube-config'}) {
          sh 'cd kubernetes/'
          sh 'kubectl cluster-info'
          sh 'kubectl create -f .\\deployment-productcatalogue.yml'
          sh 'kubectl create -f .\\deployment-stockmanager.yml'
          sh 'kubectl create -f .\\deployment-shopfront.yml'
          sh 'kubectl create -f .\\service-productcatalogue.yml'
          sh 'kubectl create -f .\\service-stockmanager.yml'
          sh 'kubectl create -f .\\service-shopfront.yml'
          sh 'kubectl create -f .\\ingress-shopfront.yml'
        }
      }
    }
  }
  always {
    // remove built docker image and prune system
    print 'Cleaning up the Docker system.'
    sh 'docker system prune -f'
  }
}
