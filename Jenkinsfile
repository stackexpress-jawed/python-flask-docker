pipeline {
    agent any
    
    options {
        // Only keep the 10 most recent builds
        buildDiscarder(logRotator(numToKeepStr:'10'))
    }
    
    triggers {
        pollSCM('* * * * *') 
    }

    parameters {
        booleanParam(
               name: 'DEBUG_MODE',
               defaultValue: false,
               description: 'Select if you want to Run in debug mode'
         )

    }

    environment {
        docker_image = ''
        latest_stage = 'Start'
    }

    stages {
        // Stage1:  Lint
        stage('Lint Dockerfile') {
            script {
                    latestStage = env.STAGE_NAME
                }
            
            steps {
                echo 'Running Dockerfile Linter...'
                
                sh 'hadolint  -V Dockerfile'
            }
        }

        // Stage3 : Build
        stage('Build Docker Image') {
            steps {
                echo 'Starting docker build...'
                script {
                    latestStage = env.STAGE_NAME
                    docker_image = docker.build("${IMAGE_NAME}:${BUILD_ID}")
                }
           }
       }

    }


    post {
        success {
            echo 'Job successful'
            
            // Send success message on Slack
            slackSend color: 'good',
            channel: "${slackChannel}",
            message: """
                Pipeline: <${env.BUILD_URL}/console|${env.JOB_NAME}>
                Git branch: `${REPO_BRANCH}`
                Build number: *${env.BUILD_DISPLAY_NAME}*
                Build status: *${currentBuild.result}*
                Total runtime: *${currentBuild.durationString}*
            """.stripIndent()
            
        }
        failure {
            echo 'Job failed'
            
            // Send alert on Slack
            slackSend color: 'danger',
            channel: "${slackChannel}",
            message: """
                Pipeline: <${env.BUILD_URL}/console|${env.JOB_NAME}>
                Git branch: `${REPO_BRANCH}`
                Build number: *${env.BUILD_DISPLAY_NAME}*
                Build status: *${currentBuild.result}*
                Failed stage *${latest_stage}*
            """.stripIndent()
            
        }
    }
  
}
