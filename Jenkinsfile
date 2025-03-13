pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                git credentialsId: 'github-pat', branch: 'main', url: 'https://github.com/csye7125-sp25-team05/helm-charts.git'
            }
        }
        stage('Release') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'github-pat', usernameVariable: 'GITHUB_USERNAME', passwordVariable: 'GITHUB_TOKEN')]) {
                    script {
                        sh "npx semantic-release"
                    }
                }
            }
        }

        
}
}
