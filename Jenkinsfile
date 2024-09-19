pipeline {
    agent any

    stages {
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

                // Use the credentials
                //def token = credentials('GITHUB_TOKEN')
                env.GITHUB_TOKEN = credentials('GITHUB_TOKEN')
                echo "Using token: ${env.GITHUB_TOKEN}" // Remove after testing!
                echo "Pull Request ID: ${prNumber}"
                echo "Other environment variables: ${env}"

                // Post to GitHub
                def response = httpRequest(
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
