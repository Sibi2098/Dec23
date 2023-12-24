pipeline{
    agent{
        lable 'docker'
    }
    options{
        timeout(time: 1, unit: 'HOURS')
    }
    triggers{
        pollSCM('* * * * *')
    }
    stages{
        stage('clean workspace'){
            steps{
                cleanWs()
            }

        }
        stage('checkout from git'){
            steps{
                git url: 'https://github.com/Sibi2098/Dec23.git',
                branch: 'main'
            }
             
        }
        stage('image build'){
            steps{
                sh "docker image build -t Sibi2098/Dec23:$BUILD_ID ."
            }
        }
        stage('trivy scan'){
            steps{
                script{
                    sh "trivy image --format json -o trivy-report.json Sibi2098/Dec23:$BUILD_ID"

                }
                publishHTML([reportName: 'Trivy Vulnerability Report', reportDir: '.', reportFiles: 'trivy-report.json', keepAll: true, alwaysLinkToLastBuild: true, allowMissing: false])
            }
        }
        stage('publish image'){
            steps{
                sh "docker image push Sibi2098/Dec23:$BUILD_ID"
            }
        }
        stage('k8s cluster'){
            sh "cd deployment/terraform && terraform init && terraform apply -auto-approve"
        }
        stage('deploy to k8s'){
            sh "kubectl apply -f deployment/k8s/deployment.yaml"
            sh """
                kubectl patch deployment netflix-app -p '{"spec":{"template":{"spec":{"containers":[{"name":"netflix-app","image":"Sibi2098/Dec23:$BUILD_ID"}]}}}}'
                """
        }
        stage('kubescape Scan') {
            steps {
                script {
                    sh "/home/ubuntu/.kubescape/bin/kubescape scan -t 40 deployment/k8s/deployment.yaml --format junit -o TEST-report.xml"
                    junit "**/TEST-*.xml"
                }
                
            }
        }    
    }
}
