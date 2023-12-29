// Jenkinsfile by Lawrence
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/liwenbo55/p3_Techscrum.fe.git'
            }
        }
        
        stage('ci'){
            steps{
                sh 'npm install'
                sh 'npm run build'
                sh 'npm run test'
            }
        }

        stage('cd'){
            steps {
                withAWS(region:'ap-southeast-2',credentials:'lawrence-jenkins-credential') {
                    withCredentials([
                    [$class: 'AmazonWebServicesCredentialsBinding', 
                        credentialsId: 'lawrence-jenkins-credential', 
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'], 
                    string(credentialsId: 'PUBLIC_CONNECTION', variable: 'PUBLIC_CONNECTION'), 
                    string(credentialsId: 'TENANTS_CONNECTION', variable: 'TENANTS_CONNECTION')]) {
                      sh 'aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com'
                      sh """
                            docker build \
                                --build-arg ENVIRONMENT="production" \
                                --build-arg NAME="techscrumapp" \
                                --build-arg PORT="8000" \
                                --build-arg API_PREFIX="/api" \
                                --build-arg AWS_REGION="ap-southeast-2" \
                                --build-arg AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}" \
                                --build-arg AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}" \
                                --build-arg ACCESS_SECRET="random" \
                                --build-arg EMAIL_SECRET="random" \
                                --build-arg FORGET_SECRET="random" \
                                --build-arg LIMITER="true" \
                                --build-arg PUBLIC_CONNECTION="${PUBLIC_CONNECTION}" \
                                --build-arg TENANTS_CONNECTION="${TENANTS_CONNECTION}" \
                                --build-arg CONNECT_TENANT="" \
                                --build-arg MAIN_DOMAIN="techscrumjr11" \
                                --build-arg STRIPE_PRIVATE_KEY="123" \
                                --build-arg STRIPE_WEBHOOK_SECRET="123" \
                                --build-arg LOGGLY_ENDPOINT="" \
                                --build-arg DEVOPS_MODE="false" \
                                -t 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-uat:latest \
                                .
                        """
                    //   sh 'docker tag techscrum-backend-ecr-uat:latest 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-uat:latest'
                    //   sh 'docker push 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-uat:latest'
                    }
                }

            // steps{
                
            // }
            }
        }


    }
        
    // post {
    //     success {
    //         emailext(
    //             to: "lawrence.wenboli@gmail.com",
    //             subject: "Jenkins Pipeline succeeded.",
    //             body: "Your Jenkins Pipeline succeeded.",
    //             attachLog: false
    //         )
    //     }

    //     failure {
    //         emailext(
    //             to: "lawrence.wenboli@gmail.com",
    //             subject: "Jenkins Pipeline failed.",
    //             body: "Your Jenkins Pipeline failed，please check logfile for more details.",
    //             attachLog: true
    //         )
    //     }
    // }
}
