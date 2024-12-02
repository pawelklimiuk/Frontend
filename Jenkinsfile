def imageName="pawelk123456/frontend"
def dockerTag=""
def dockerRegistry=""
def registryCredentials="dockerhub"


pipeline {
    agent any 
    //  {
    //     label 'agent'
    // }
    
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
                    // dockerTag = "RC-${env.BUILD_ID}"
                    dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"
                    applicationImage = docker.build("$imageName:$dockerTag")
                }
            }
        }
        
        stage ('Wypychamy image do docker registry') {
            steps {
                script {
                    docker.withRegistry("$dockerRegistry", "$registryCredentials") {
                        applicationImage.push()
                        applicationImage.push('latest')
                    }
                }
            }
        }
        
        stage ('Push to Repo') {
            steps {
                dir('ArgoCD') {
                    withCredentials([gitUsernamePassword(credentialsId: 'git', gitToolName: 'Default')]) {
                        git branch: 'main', url: 'https://github.com/Panda-Academy-Core-2-0/ArgoCD.git'
                        sh """ cd frontend
                        sed -i "s#$imageName.*#$imageName:$dockerTag#g" deployment.yaml
                        git commit -am "Set new $dockerTag tag."
                        git push origin main
                        """
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
            //build job: 'app_of_apps', parameters: [ string(name: 'frontendDockerTag', value: "$dockerTag")], wait: false
        }
    }
}
