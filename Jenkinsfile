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
        stage('Tests'){
            steps {
                dir('app') {
                    sh './gradlew test'
                }
            }
            post {
                always {
                    junit(testResults: 'app/build/test-results/test/*xml', allowEmptyResults: true)
                    jacoco(
                        classPattern: 'app/build/classes/java/main', 
                        execPattern: 'app/build/jacoco/*.exec',
                        sourcePattern: 'app/src/main/java/com/example/restservice')
                }
            }
        }
        stage('Image generation'){
            steps {
                sh 'VERSION_TAG=1.0.${BUILD_NUMBER} docker-compose build'
                sh 'VERSION_TAG=1.0.${BUILD_NUMBER} docker-compose push'
                sh 'docker-compose build'
                sh 'docker-compose push'
                sh 'git tag 1.0.${BUILD_NUMBER}'
                sshagent(['github-ssh']) {
                    sh 'git push git@github.com:${GIT_PATH}.git --tags'
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