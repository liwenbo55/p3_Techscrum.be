// Jenkinsfile by Lawrence
pipeline {
    agent any

    parameters {
        // Choose an environment to deploy back-end.
        choice(choices: ['dev', 'uat', 'prod'], name: 'Environment', description: 'Please choose an environment to deploy back-end.')
    }

    environment {
        AWS_REGION  = "ap-southeast-2"
        HOSTED_ZONE = credentials('HOSTED_ZONE_NAME')
        ECR_REGISTRY_URL = "364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-${params.Environment}"
        PROJECT_ENV = "${params.Environment}"
        // MAIN_DOMAIN = "${params.Environment}.${HOSTED_ZONE}"
        BACKEND_HEALTHCHECK_URL = "backend.${params.Environment}.${HOSTED_ZONE}/api/v2/healthcheck"
        
    }

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
                withAWS(credentials: 'lawrence-jenkins-credential', region: 'ap-southeast-2') {
                    script {
                        
                        sh "echo 'Backend (${PROJECT_ENV}) is deploying...'"
                        
                        // Remove unused build cache
                        sh 'docker builder prune -f'
                        
                        // Login to AWS ECR
                        // sh 'aws ecr get-login-password --region ap-southeast-2 | docker login --username AWS --password-stdin 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com'
                        sh "aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY_URL}"

                        
                        // if (params.Environment == 'prod') {
                        //     MAIN_DOMAIN = "${HOSTED_ZONE}"
                        // } else {
                        //     MAIN_DOMAIN = "${params.Environment}.${HOSTED_ZONE}"
                        // }
                        // sh "echo 'main domain: ${env.MAIN_DOMAIN}'" 
                        
                        // Build docker image                                  
                        sh "docker build -t ${ECR_REGISTRY_URL}:latest ."
                        
                        sh "echo 'main domain: ${HOSTED_ZONE}'" 

                        // sh "echo 'main domain: ${MAIN_DOMAIN}'" 

                        // Push docker image to AWS ECR
                        // sh 'docker push 364250634199.dkr.ecr.ap-southeast-2.amazonaws.com/techscrum-backend-ecr-uat:latest'
                        sh 'docker push ${ECR_REGISTRY_URL}:latest'
                        

                        // ///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
                        // // Fetch task-definition
                        // sh "aws ecs describe-task-definition --task-definition techscrum-ecs-task-definition-${PROJECT_ENV} --query 'taskDefinition' > task_definition.json --region ap-southeast-2" 
                        
                        // // Generate a new task definition
                        // def new_pushed_image = "${ECR_REGISTRY_URL}:latest"
                        // sh """
                        //     jq --arg new_image "${new_pushed_image}" \
                        //        'del(.taskDefinitionArn, .revision, .status, .requiresAttributes, .compatibilities, .registeredAt, .registeredBy) | .containerDefinitions[0].image = \$new_image' \
                        //        task_definition.json > new_task_definition.json
                        //    """
                        
                        // // Register new task definition
                        //   sh "aws ecs register-task-definition --cli-input-json file://new_task_definition.json --region ${AWS_REGION}"
                        // ///////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
                        
                        // Update ECS service:
                          sh """
                             aws ecs update-service \
                                    --cluster 'techscrum-ecs-cluster-${PROJECT_ENV}' \
                                    --service 'techscrum-ecs-service-${PROJECT_ENV}' \
                                    --task-definition 'techscrum-ecs-task-definition-${PROJECT_ENV}' \
                                    --region ${AWS_REGION} \
                                    --force-new-deployment
                             """
                        }
                }
            }
        }
    }
        
    post {
        success {
            echo "Backend Healthcheck url: ${BACKEND_HEALTHCHECK_URL}"
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Backend CI/CD pipeline (${PROJECT_ENV} environment) succeeded.",
                body: "Jenkins CI/CD Pipeline succeeded.\n\nEnvironment: ${PROJECT_ENV}.",
                attachLog: false
            )
        }

        failure {
            emailext(
                to: "lawrence.wenboli@gmail.com",
                subject: "Backend cicd pipeline (${PROJECT_ENV} environment) failed.",
                body: "Jenkins Pipeline failed.\n\nEnvironment: ${PROJECT_ENV}.\n\nPlease check logfile for more details.",
                attachLog: true
            )
        }
    }
}
