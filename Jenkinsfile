def imageName="192.168.44.44:8082/docker_registry/frontend"
def dockerTag=""
def dockerRegistry="https://192.168.44.44:8082"
def registryCredentials="artifactory"


pipeline {
    agent {
         label 'agent'
    }
    
    environment {
        PIP_BREAK_SYSTEM_PACKAGES = 1
        scannerHome = tool 'SonarQube'
    }
    
    stages {
        stage('Pobierz kod z repo') {
            steps {
                checkout scm
            }
        }
        
        stage('Wykonaj testy') {
            steps {
                sh "pip3 install -r requirements.txt"
                sh "python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml"
            }
        }
        
        stage('Anzaliza Sonarqube') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        } 
        
        stage('Budujemy kontener z aplikacja') {
            steps {
                script {
                    dockerTag = "RC-${env.BUILD_ID}"
                    applicationImage = docker.build("$imageName:$dockerTag")
                }
            }
        }
        
        stage ('Wypychamy image do artifactory do docker registry') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
    }
    
    post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
        success {
            build job: 'app_of_apps', parameters: [ string(name: 'frontendDockerTag', value: "$dockerTag")], wait: false
        }
    }
}
