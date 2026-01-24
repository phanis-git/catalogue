pipeline {
    agent { 
        label 'AGENT-1'
    }
    environment {
        appVersion = ""
        project = "roboshop"
        component = "catalogue"
    }
    stages {
        stage('Read Version'){
            steps {
                script {
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                }
            }
        }
        stage('Install Dependencies'){
            steps {
                sh """ 
                    npm install
                """
            }
        }
         stage('Unit Testing'){
            steps {
                sh """ 
                    npm test
                """
            }
        }
        // here we should select the scanner tool and send the analysis to server
        stage('Sonar scan'){
            environment {
                def scannerHome = tool 'sonar-8.0'
            }
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 1, unit: 'HOURS') {
                    // Wait for the quality gate status
                    // abortPipeline: true will fail the Jenkins job if the quality gate is 'FAILED'
                    waitForQualityGate abortPipeline: true 
                }
            }
        }

        stage('Build Image') {
            steps {
                withAWS(region:'us-east-1',credentials:'nameOfSystemCredentials') {
                    aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 658038453909.dkr.ecr.us-east-1.amazonaws.com

                    docker build -t roboshop/catalogue:latest 658038453909.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .

                    docker push 658038453909.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion}
                }
                // sh """ 
                //     docker build -t catalogue:${appVersion} . 
                // """
            }
        }
    }
}


// RHEL-9-DevOps-Practice