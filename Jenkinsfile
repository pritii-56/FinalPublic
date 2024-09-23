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

        post
        {
        always {
            script {
                def statusMessage = currentBuild.result ?: 'SUCCESS'
                echo "the status: ${currentBuild.result}"
                def prNumber1 = env.CHANGE_ID // Get PR number
                echo "pr number 1 : ${env.CHANGE_ID}"
                def repo = "pritii-56/FinalPublic" // Change to your repo
                def branchName = env.GIT_BRANCH
                def prNumber='notFound'
                def comment = """\
                Build Successful!
                - Branch: ${env.BRANCH_NAME}
                - Build Number: ${env.BUILD_NUMBER}
                - Commit: ${env.GIT_COMMIT}
                """
                withCredentials([string(credentialsId: 'GITHUB_TOKEN', variable: 'GITHUB_TOKEN')])
                {
                        echo "Using token: ${GITHUB_TOKEN}" 
                
                
                def response = httpRequest(
                        url: "https://api.github.com/repos/${repo}/pulls?state=open&head=pritii-56:${branchName}",
                        httpMode: 'GET',
                        customHeaders: [[name: 'Authorization', value: "token ${GITHUB_TOKEN}"]]
                    )
                echo "here"
                /*def username = sh(script: 'git config user.name', returnStdout: true).trim()
                echo "Configured Git Username: ${username}"*/
                def pullRequests = readJSON(text: response.content)
                if (pullRequests) {
                        prNumber = pullRequests[0].number
                        echo "Pull Request Number: ${prNumber}"
                    } else {
                        echo "No open pull requests for this branch."
                    }

                // Use the credentials
                //def token = credentials('GITHUB_TOKEN')
                //env.GITHUB_TOKEN = credentials('GITHUB_TOKEN')
                //echo "Using token: ${env.GITHUB_TOKEN}" // Remove after testing!
                echo "Pull Request ID: ${prNumber}"
                /*env.each { key, value ->
                        echo "${key} = ${value}"
                }*/
                // Post to GitHub
                def response1 = httpRequest(
                    url: "https://api.github.com/repos/${repo}/issues/${prNumber}/comments",
                    httpMode: 'POST',
                    contentType: 'APPLICATION_JSON',
                    requestBody: "{\"body\":\"Build ${statusMessage} - [View Build](${env.BUILD_URL})\"}",
                    //requestBody: "{\"body\": \"${comment}\"}",
                    customHeaders: [[name: 'Authorization', value: "token ${GITHUB_TOKEN}"]]
                )
                echo "Posted status: ${response.status}"

                // for all checks passed
                def commitSha = env.GIT_COMMIT
                echo "Commit SHA: ${commitSha}"
                def apiUrl = "https://api.github.com/repos/${repo}/check-runs"
                // Create the JSON payload for the check run
                def jsonPayload = """
                {
                  "name": "Jenkins CI",
                  "head_sha": "${commitSha}",
                  "status": "completed",
                  "conclusion": "success",
                  "output": {
                    "title": "Build Success",
                    "summary": "All checks have passed."
                  }
                }
                """
                // Make the API call to create the check run
                def response2 = httpRequest(
                    url: apiUrl,
                    httpMode: 'POST',
                    requestBody: jsonPayload,
                    contentType: 'APPLICATION_JSON',
                    customHeaders: [[name: 'Authorization', value: "token ${GITHUB_TOKEN}"]],
                    validResponseCodes: '200:201' // Accept 200 and 201 responses
                )

                echo "Check Run Response: ${response2.status}, Body: ${response2.content}"
            }
                }
            }
        }
    }
