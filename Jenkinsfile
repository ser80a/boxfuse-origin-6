pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
    timeout(time: 30, unit: 'MINUTES')
  }

  parameters {
    string(
      name: 'BRANCH_NAME',
      defaultValue: 'fe/dev',
      description: 'Branch to deploy (fe/dev, fe/qa, master)'
    )
  }

  environment {
    APP_NAME      = 'boxfuse-sample-java-war-hello'
    WAR_NAME      = 'boxfuse-sample-java-war-hello.war'
    WAR_SOURCE    = 'target/hello-1.0.war'
    WAR_TARGET    = "target/${WAR_NAME}"
    GIT_URL       = 'https://github.com/RAJANI9/boxfuse-sample-java-war-hello.git'
    CONTEXT_PATH  = "./${APP_NAME}"
    SLACK_CHANNEL = '7-30am-cloud-devops-batch'
  }

  stages {

    stage('Checkout') {
      steps {
        script {
          echo "Checking out branch: ${params.BRANCH_NAME}"
          checkout([
            $class: 'GitSCM',
            branches: [[name: params.BRANCH_NAME]],
            userRemoteConfigs: [[url: env.GIT_URL]]
          ])
        }
      }
    }

    stage('Test') {
      steps {
        echo 'Running tests...'
        sh 'mvn -B -q test'
      }
    }

    stage('Build') {
      steps {
        echo 'Building package...'
        sh 'mvn -B clean package'
        sh "mv ${env.WAR_SOURCE} ${env.WAR_TARGET}"
      }
      post {
        success {
          echo "Build complete: ${env.WAR_TARGET}"
        }
        failure {
          echo 'Build failed.'
        }
      }
    }

    stage('Deploy to Dev') {
      when {
        expression { params.BRANCH_NAME == 'fe/dev' }
      }
      steps {
        script {
          deployToTomcat(
            tomcatIP   : '13.201.47.166',
            username   : 'tomcat',
            password   : 'tomcat',
            tomcatURL  : 'http://13.201.47.166:8080/manager/text',
            contextPath: env.CONTEXT_PATH,
            environment: 'Dev',
            warFile    : env.WAR_TARGET
          )
        }
      }
    }

    stage('Deploy to QA') {
      when {
        expression { params.BRANCH_NAME == 'fe/qa' }
      }
      steps {
        script {
          deployToTomcat(
            tomcatIP   : '13.201.47.166',
            username   : 'tomcat',
            password   : 'tomcat',
            tomcatURL  : 'http://13.201.47.166:8080/manager/text',
            contextPath: env.CONTEXT_PATH,
            environment: 'QA',
            warFile    : env.WAR_TARGET
          )
        }
      }
    }

    stage('Deploy to Prod') {
      when {
        expression { params.BRANCH_NAME == 'master' }
      }
      steps {
        input message: 'Do you want to proceed to PROD?', ok: 'Proceed'
        script {
          deployToTomcat(
            tomcatIP   : '65.2.187.108',
            username   : 'tomcat',
            password   : 'tomcat',
            tomcatURL  : 'http://65.2.187.108:8080/manager/text',
            contextPath: env.CONTEXT_PATH,
            environment: 'Prod',
            warFile    : env.WAR_TARGET
          )
        }
        slackSend channel: env.SLACK_CHANNEL, message: 'Deployment to PROD has been approved by manager.'
      }
    }
  }

  post {
    success {
      echo 'Pipeline finished successfully.'
    }
    failure {
      echo 'Pipeline failed.'
    }
    always {
      echo "Branch deployed (if any): ${params.BRANCH_NAME}"
    }
  }
}

/**
 * Reusable deployment function.
 * Uses Tomcat Manager "deploy" endpoint with update=true.
 */
def deployToTomcat(Map cfg) {
  def warFile = cfg.warFile ?: "target/${env.WAR_NAME}"
  echo "Deploying ${warFile} to ${cfg.environment} at ${cfg.tomcatURL} (context: ${cfg.contextPath})"

  sh """
    curl -sS -f -u ${cfg.username}:${cfg.password} \
      --upload-file ${warFile} \
      ${cfg.tomcatURL}/deploy?path=${cfg.contextPath}&update=true

    echo "Deployment to ${cfg.environment} server completed."
  """
}