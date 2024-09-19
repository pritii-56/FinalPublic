pipeline {
    agent any

    stages {

         stage('Checkout Code') {
            steps {
                checkout scm
            }
        }
        stage('Init') {
            steps {
                echo 'Initializing..'
                echo "Running ${env.BUILD_ID} on ${env.JENKINS_URL}"
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
                echo 'Running pytest..'
            }
        }
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
    }
        post {
        always {
            script {
                def statusMessage = currentBuild.result ?: 'SUCCESS'
                def prNumber = env.CHANGE_ID // Get PR number
                def repo = "pritii-56/FinalPublic" // Change to your repo
                def branchName = env.GIT_BRANCH
                def response = httpRequest(
                        url: "https://api.github.com/repos/${repo}/pulls?state=open&head=pritii-56:${branchName}",
                        httpMode: 'GET',
                        customHeaders: [[name: 'Authorization', value: "token ${GITHUB_TOKEN}"]]
                    )
                def username = sh(script: 'git config user.name', returnStdout: true).trim()
                echo "Configured Git Username: ${username}"
                def pullRequests = readJSON(text: response.content)
                if (pullRequests) {
                        def prNumber = pullRequests[0].number
                        echo "Pull Request Number: ${prNumber}"
                    } else {
                        echo "No open pull requests for this branch."
                    }

                // Use the credentials
                //def token = credentials('GITHUB_TOKEN')
                //env.GITHUB_TOKEN = credentials('GITHUB_TOKEN')
                //echo "Using token: ${env.GITHUB_TOKEN}" // Remove after testing!
                echo "Pull Request ID: ${prNumber}"
                env.each { key, value ->
                        echo "${key} = ${value}"
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')]) {
                        echo "Using token: ${GITHUB_TOKEN}" 
                }
                }

                // Post to GitHub
                def response1 = httpRequest(
                    url: "https://api.github.com/repos/${repo}/issues/${prNumber}/comments",
                    httpMode: 'POST',
                    contentType: 'APPLICATION_JSON',
                    requestBody: "{\"body\":\"Build ${statusMessage} - [View Build](${env.BUILD_URL})\"}",
                    customHeaders: [[name: 'Authorization', value: "token ${env.GITHUB_TOKEN}"]]
                )
                echo "Posted status: ${response.status}"
            }
        }
    }
}
