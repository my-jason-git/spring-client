pipeline {
  agent any
  tools { 
    maven 'maven'
  }
  environment {
    LOGIN_URL = 'https://api.ocpuat.pccw.com'
    LOGIN_PORT = '6443'
    PROJECT = 'test'
  }  
  stages {
    stage ('Initialize') {
      steps {
        sh '''
          echo "PATH = ${PATH}"
          echo "M2_HOME = ${M2_HOME}"
        ''' 
      }
    }
    stage('Login') {
      steps {
        withCredentials(bindings: [usernamePassword(
          		  credentialsId: 'sha256~TtOOSsM-CB_T9h6IVOPZAkw9abPlPClVx4VWuMYlI8U', 
          		  usernameVariable: 'USERNAME',
          		  passwordVariable: 'PASSWORD',
          		)]) {
          sh "oc login ${env.LOGIN_URL}:${env.LOGIN_PORT} --token=${PASSWORD}"
        }

      }
    }
    stage('Delete Project') {
      steps {
        sh 'oc delete project springclient-ns'
      }
    }
    stage('Maven Build') {
      steps {
        echo 'Build jar file'
        sh 'mvn clean install -DskipTests=true'
      }
    }
    stage('Run Unit Tests') {
      steps {
        echo 'Run unit tests'
        sh 'mvn test'
      }
    }
    stage('Create Project') {
      steps {
        echo 'Create Project'
        script {
          sh 'oc new-project springclient-ns'
          sh 'oc project springclient-ns'
          echo "Using project: ${openshift.project()}"
        }

      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploy application'
        script {
          sh 'oc new-app --name springclient \'registry.access.redhat.com/redhat-openjdk-18/openjdk18-openshift:1.6~https://github.com/remkohdev/spring-client\' --strategy=source --allow-missing-images --build-env=\'JAVA_APP_JAR=hello.jar\''
        }

      }
    }
    stage('Expose') {
      steps {
        echo 'Expose Route'
        script {
          sh 'oc expose svc/springclient'
        }

      }
    }
  }
}
