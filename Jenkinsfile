
pipeline {
    agent any

    options {
        skipDefaultCheckout()
    }

    environment {
        COMMITTED_BY = 'Unknown'
        DEPLOY_BRANCH = 'subbranch'
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs(deleteDirs: true)
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'subbranch',
                    credentialsId: 'sruthi.d',
                    url: 'https://github.com/deekondasruthi/website.git'

                script {
                    echo "Checking Git commit details..."

                    sh 'git log -1 --format=fuller'

                    def author = sh(
                        script: 'git log -1 --format=%an',
                        returnStdout: true
                    ).trim()

                    def authorEmail = sh(
                        script: 'git log -1 --format=%ae',
                        returnStdout: true
                    ).trim()

                    def committer = sh(
                        script: 'git log -1 --format=%cn',
                        returnStdout: true
                    ).trim()

                    env.COMMITTED_BY = author ?
                        "${author} <${authorEmail}>" :
                        (committer ?: 'Unknown')

                    echo "Git Author: ${author}"
                    echo "Git Author Email: ${authorEmail}"
                    echo "Git Committer: ${committer}"
                    echo "FINAL COMMITTED BY: ${env.COMMITTED_BY}"
                    echo "BRANCH: ${env.DEPLOY_BRANCH}"
                }
            }
        }

        stage('Check Project Structure') {
            steps {
                sh '''
                    echo "Current directory:"
                    pwd

                    echo "Repository files:"
                    ls -la

                    echo "Searching for package.json:"
                    find . -name package.json -not -path '*/node_modules/*' -print
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install --force'
            }
        }

        stage('Build Angular') {
            steps {
                sh 'npx ng build'
            }
        }
    }

    post {

        success {
            emailext(
                subject: "Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>Build Status: <b style="color:green;">SUCCESS</b></p>

                    <p><b>Committed By:</b> ${env.COMMITTED_BY}</p>

                    <p>Application API Base URL: https://staging-cable.farginconnect.com</p>

                    <p>Using the above base URL, you can verify UI deployment and test all frontend features.</p>

                    <p><b>Branch:</b> ${env.DEPLOY_BRANCH}</p>

                    <br/>
                    <p>Regards,</p>
                    <p>DevOps Team.</p>

                    <p>
                        <img src="https://babujiventures.in/assets/img/clients/baabuji-logo-1-cropped.png"
                             alt="Baabuji Logo" style="height:60px;">
                    </p>
                """,
                to: 'sruthi.d@babujiventures.in',
                mimeType: 'text/html'
            )
        }

        failure {
            emailext(
                subject: "Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                body: """
                    <p>Build Status: <b style="color:red;">FAILED</b></p>

                    <p><b>Committed By:</b> ${env.COMMITTED_BY}</p>

                    <p><b>Branch:</b> ${env.DEPLOY_BRANCH}</p>

                    <p>Please review the attached build log file to identify the issue.</p>

                    <br/>
                    <p>Regards,</p>
                    <p>DevOps Team.</p>

                    <p>
                        <img src="https://babujiventures.in/assets/img/clients/baabuji-logo-1-cropped.png"
                             alt="Baabuji Logo" style="height:60px;">
                    </p>
                """,
                to: 'sruthi.d@babujiventures.in',
                attachLog: true,
                mimeType: 'text/html'
            )
        }
    }
}
