pipeline {
    agent any
    environment{
        GIT_USER = 'EricHeresi'
        GIT_PATH = 'ericheresi/hello-springrest'
    }
    options{
        timestamps()
         ansiColor('xterm')
    }
    stages {
        stage('Git Login'){
            steps {
                withCredentials([string(credentialsId: 'github-token', variable: 'GIT_TOKEN')]) {
                    sh 'echo $GIT_TOKEN | docker login ghcr.io -u ${GIT_USER} --password-stdin'
                }
            }
        }
        stage('Trivy'){
            steps{
                sh 'mkdir -p trivy'
                sh 'trivy repo -f json -o trivy/results_repo.json https://github.com/${GIT_PATH}'
                sh 'trivy fs -f json -o trivy/results_fs.json .'
            }
        }
        stage('PMD Check'){
            steps {
                sh './gradlew check'
            }
            post {
                always {
                    recordIssues(tools: [pmdParser(pattern: 'build/reports/pmd/*.xml')])
                }
            }
        }
        stage('Tests'){
            steps {
                sh './gradlew test'
            }
            post {
                always {
                    junit(testResults: 'build/test-results/test/*xml', allowEmptyResults: true)
                    jacoco(
                        classPattern: 'build/classes/java/main', 
                        execPattern: 'build/jacoco/*.exec',
                        sourcePattern: 'src/main/java/com/example/restservice')
                }
            }
        }
        stage('Image generation'){
            steps {
                sh 'VERSION_TAG=1.0.${BUILD_NUMBER} docker-compose build'
                sh 'VERSION_TAG=1.0.${BUILD_NUMBER} docker-compose push'
                sh 'docker-compose build'
                sh 'trivy image -f json -o trivy/results_img.json ghcr.io/${GIT_PATH}/aws_springrest:latest'
                sh 'docker-compose push'
                sh 'git tag 1.0.${BUILD_NUMBER}'
                sshagent(['github-ssh']) {
                    sh 'git push git@github.com:${GIT_PATH}.git --tags'
                }
            }
            post{
                always {
                    recordIssues(tools: [trivy(pattern: 'trivy/*.json')])
                }
            }
        }
        stage("Infrastructure"){
            steps {
                withAWS(credentials: 'aws-access-key', region: 'eu-west-1') {
                    dir("eb") {
                        sh 'eb deploy hello-springrest'
                    }
                }
            }
        }
    }
}