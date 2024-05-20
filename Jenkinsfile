pipeline {
    agent any
    tools{
        nodejs 'node'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('git checkout'){
            steps{
                git branch: 'master', url: 'https://github.com/jaddusantoshkumar/express.git'
            }
        }
        stage('Building docker image and pushing into registry'){
            steps{
                script{
                    withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){
                        sh '''
                        docker build -t jaddusantoshkumar/example:latest .
                        docker push jaddusantoshkumar/example:latest
                        '''
                    }
                }
            }
        }
        stage('Deploying the application'){
            steps{
                sh '''
                kubectl apply -f deployment.yaml
                kubectl apply -f service.yaml
                '''
            }
        }
    }
}
