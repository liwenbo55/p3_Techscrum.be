// Jenkinsfile by Lawrence
pipeline {
    agent any

    stages {
        stage('Git checkout') {
            steps{
                // Get source code from a GitHub repository
                git branch:'main', url:'https://github.com/liwenbo55/p3_Techscrum.be.git'
            }
        }
        
        // stage('ci'){
        //     steps{
        //         script{
        //             sh 'npm install'
        //             sh 'npm run build'
        //             sh 'npm run test'
        //         }
        //     }
        // }

        stage('cd'){
            steps {
                script {
                    withCredentials([
                        [$class: 'AmazonWebServicesCredentialsBinding', 
                            credentialsId: 'lawrence-jenkins-credential', 
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID', 
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'], 
                        string(credentialsId: 'PUBLIC_CONNECTION', variable: 'PUBLIC_CONNECTION'), 
                        string(credentialsId: 'TENANTS_CONNECTION', variable: 'TENANTS_CONNECTION')]) {
                          // sh 'aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com'
                          // sh '''
                          //       docker build \
                          //           --build-arg ENVIRONMENT="production" \
                          //           --build-arg NAME="techscrumapp" \
                          //           --build-arg PORT="8000" \
                          //           --build-arg API_PREFIX="/api" \
                          //           --build-arg AWS_REGION="ap-southeast-2" \
                          //           --build-arg AWS_ACCESS_KEY_ID="${AWS_ACCESS_KEY_ID}" \
                          //           --build-arg AWS_SECRET_ACCESS_KEY="${AWS_SECRET_ACCESS_KEY}" \
                          //           --build-arg ACCESS_SECRET="random" \
                          //           --build-arg EMAIL_SECRET="random" \
                          //           --build-arg FORGET_SECRET="random" \
                          //           --build-arg LIMITER="true" \
                          //           --build-arg PUBLIC_CONNECTION="${PUBLIC_CONNECTION}" \
                          //           --build-arg TENANTS_CONNECTION="${TENANTS_CONNECTION}" \
                          //           --build-arg CONNECT_TENANT="" \
                          //           --build-arg MAIN_DOMAIN="techscrumjr11" \
                          //           --build-arg STRIPE_PRIVATE_KEY="123" \
                          //           --build-arg STRIPE_WEBHOOK_SECRET="123" \
                          //           --build-arg LOGGLY_ENDPOINT="" \
                          //           --build-arg DEVOPS_MODE="false" \
                          //           -t 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-uat:latest \
                          //           .
                          //    '''
                          // sh 'docker push 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-uat:latest'
                          sh "aws ecs describe-task-definition --task-definition techscrum-ecs-task-definition-uat --query 'taskDefinition' > task_definition.json --region ap-southeast-2" 
                          sh '''
                                jq --arg new_image "364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-uat:latest" \
                                    'del(.taskDefinitionArn, .revision, .status, .requiresAttributes, .compatibilities, .registeredAt, .registeredBy) | .containerDefinitions[0].image = $new_image' \
                                    task_definition.json > new_task_definition.json
                             '''
                          sh 'aws ecs register-task-definition --cli-input-json file://new_task_definition.json'
                          sh '''
                             aws ecs update-service \
                                    --cluster techscrum-ecs-cluster-uat \
                                    --service techscrum-ecs-service-uat \
                                    --task-definition techscrum-ecs-task-definition-uat \
                                    --force-new-deployment
                             '''
                        }                    
                }
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
