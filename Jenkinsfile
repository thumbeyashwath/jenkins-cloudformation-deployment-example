pipeline {
    agent {label 'Linux'}
    
    //tools {
    //    git 'LinuxGit' 
    //    }
    environment {
        linuxgit = tool name: 'LinuxGit', type: 'git'
    }
    
    parameters {
    string(name: 'STACK_NAME', defaultValue: 's3-stack-by-CICD', description: 'Enter the CloudFormation Stack Name.')
    string(name: 'PARAMETERS_FILE_NAME', defaultValue: 'example-stack-parameters.properties', description: 'Enter the Parameters File Name (Must contain file extension type *.properties)')
    string(name: 'TEMPLATE_NAME', defaultValue: 'S3-Bucket.yaml', description: 'Enter the CloudFormation Template Name (Must contain file extension type *.yaml)')
    credentials(name: 'CFN_TCC_CREDENTIALS_ID', defaultValue: '', description: 'AWS Account Role.', required: true)
    choice(
      name: 'REGION',
      choices: [
          ' ',
          'us-east-1',
          'us-east-2'
          ],
      description: 'AWS Account Region'
    )
    choice(
      name: 'ACTION',
      choices: ['create-changeset', 'execute-changeset', 'deploy-stack', 'delete-stack'],
      description: 'CloudFormation Actions'
    )
    booleanParam(name: 'TOGGLE', defaultValue: false, description: 'Are you sure you want to perform this action?')
  }
  
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('git checkout') {
            steps {
                sh "${linuxgit} clone -b dev git@github.com:thumbeyashwath/jenkins-cloudformation-deployment-example.git"
                sh "ls -lrt jenkins-cloudformation-deployment-example/scripts/"
                sh "pwd"
        }
    }
    stage('check version') {
      steps {
        ansiColor('xterm') {
          /*container("jenkins-agent") {*/
            sh 'aws --version'
            //sh 'aws sts get-caller-identity'
          }
        }
      }
      
      
      stage('action') {
      when {
        expression { params.ACTION == 'create-changeset' || params.ACTION == 'execute-changeset' || params.ACTION == 'deploy-stack' || params.ACTION == 'delete-stack'}
      }
      steps {
        ansiColor('xterm') {
          script {
            if (!params.TOGGLE) {
                currentBuild.result = 'ABORTED' //If you do not set the toggle flag to true before executing the build action, it will automatically abort the pipeline for any action.
            } else {
                if (params.ACTION == 'create-changeset') {
                    env.CHANGESET_MODE = false
                } else {
                    env.CHANGESET_MODE = true
                }
            }
          }
        }
      }
    }


    stage('stack-execution') {
        when {
        expression { params.ACTION == 'deploy-stack' || params.ACTION == 'execute-changeset' }
      }
    steps {
        withCredentials([[
            $class: 'AmazonWebServicesCredentialsBinding',
            credentialsId: '${CFN_TCC_CREDENTIALS_ID}',
            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        ]]) {
            sh 'echo AWS_ACCESS_KEY_ID is ${AWS_ACCESS_KEY_ID} '
            sh 'echo AWS_SECRET_ACCESS_KEY is ${AWS_SECRET_ACCESS_KEY}'
            sh 'AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} AWS_DEFAULT_REGION=${REGION} jenkins-cloudformation-deployment-example/scripts/deploy-stack-dev-describe.sh ${STACK_NAME} ${PARAMETERS_FILE_NAME} ${TEMPLATE_NAME} ${CHANGESET_MODE} ${REGION}'
            
            }
        }
    }
    
    stage('create-changeset') {
      when {
        expression { params.ACTION == 'create-changeset' }
      }
      steps {
        ansiColor('xterm') {
         // container("jenkins-agent") {
            withCredentials([[
              $class: 'AmazonWebServicesCredentialsBinding',
              credentialsId: '${CFN_TCC_CREDENTIALS_ID}',
              accessKeyVariable: 'AWS_ACCESS_KEY_ID',
              secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                sh 'AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} AWS_DEFAULT_REGION=${REGION} jenkins-cloudformation-deployment-example/scripts/deploy-stack-dev.sh ${STACK_NAME} ${PARAMETERS_FILE_NAME} ${TEMPLATE_NAME} ${CHANGESET_MODE} ${REGION}'
            }
          }
        //}
      }
    }

    stage('delete-stack') {
      when {
        expression { params.ACTION == 'delete-stack' }
      }
      steps {
        ansiColor('xterm') {
         // container("jenkins-agent") {
            withCredentials([[
              $class: 'AmazonWebServicesCredentialsBinding',
              credentialsId: '${CFN_TCC_CREDENTIALS_ID}',
              accessKeyVariable: 'AWS_ACCESS_KEY_ID',
              secretKeyVariable: 'AWS_SECRET_ACCESS_KEY']]) {
                sh 'AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID} AWS_SECRET_ACCESS_KEY=${AWS_SECRET_ACCESS_KEY} AWS_DEFAULT_REGION=${REGION} jenkins-cloudformation-deployment-example/scripts/delete-stack-dev.sh ${STACK_NAME} ${REGION}'
            }
          //}
        }
      }
    }
    
      
}
}