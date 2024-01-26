pipeline {
    agent any
    tools {nodejs "Node 21.6"}
    environment {
        version = ''
        newVersion = ''
        newVersionTag = ''
        repoUrl = 'https://github.com/keithstric/jenkins-experiment.git'
        repoUrlPath = 'github.com/keithstric/jenkins-experiment.git'
        repoBranch = 'main'
    }
    stages {
        stage('Node Install') {
            steps {
                echo 'Installing dependencies...'
                git branch: "${repoBranch}", credentialsId: '2e31314d-3846-45a9-b554-76317c61b288', url: "${repoUrl}"
                script {
                    version = readJSON(file: './package.json').version
                }
            }
        }
        stage('Bump version') {
            steps {
                echo 'Starting the build...'
                sh 'npm run bump-version:patch'
                script {
                    newVersion = readJSON(file: './package.json').version
                    newVersionTag = "v${newVersion}"
                }
            }
        }
        stage('Tagging and Updating Branch with new version') {
            steps {
                echo "Tagging git branch with new version tag ${newVersionTag}"
                withCredentials([usernamePassword(credentialsId: '2e31314d-3846-45a9-b554-76317c61b288', passwordVariable: 'GITHUB_PW', usernameVariable: 'GITHUB_USER')]) {
                    sh """
                        git config --global credential.username $GITHUB_USER
                        git config --global credential.helper '!f() { echo password=$GITHUB_PW; }; f'
                        git config --global user.name 'Jenkins'
                        git config --global user.email 'keithstric@gmail.com'
                        git add ./package.json
                        git commit -m 'Jenkins - updated build version to ${newVersion}'
                        git push --set-upstream origin main
                    """
                }
            }
        }
        stage('Cleanup Local Jenkins environment') {
            steps {
                echo 'Cleaning up environment variables'
                script {
                    version = ''
                    newVersion = ''
                    newVersionTag = ''
                    repoUrl = ''
                    repoUrlPath = ''
                    repoBranch = ''
                }
            }
        }
    }
}
