node {

   stage 'Checkout'
   git 'https://github.com/omarlari/aws-container-sample-app.git'

   stage 'Build Dockerfile'
   docker.build('hello')

   stage 'Push to ECR'
   sh("eval \$(aws ecr get-login --no-include-email | sed 's|https://||')")
   docker.withRegistry('https://016309051001.dkr.ecr.us-east-1.amazonaws.com', 'ecr:us-east-1:ecr-credentials') {
       docker.image('hello').push('${BUILD_NUMBER}')
   }

   stage 'update application'

   parallel(
        ecs: {node {
        docker.image('awscli').inside{
            git 'https://github.com/omarlari/aws-container-sample-app.git'
            sh 'sed -i s/REPO/${ECS_REPO}/g task-definition-hello.json'
            sh 'sed -i s/BUILD/${BUILD_NUMBER}/g task-definition-hello.json'
            sh 'ecs register-task-definition --cli-input-json file://task-definition-hello.json --family ${TASK_DEF} --region ${REGION}'
        }
        }},
        kubernetes: { node {
        docker.image('kubectl').inside("--volume=/home/ec2-user/.kube:/config/.kube"){
            sh 'set image deployment/${K8S_DEPLOYMENT} movies=${ECS_REPO}/movies:${BUILD_NUMBER}'
            }
        }},
        swarm: { node {
            sh "echo deploying to swarm"
        }}
   )


   stage 'update ECS service'
   docker.image('awscli').inside{
       sh 'aws ecs update-service --cluster ${ECS_CLUSTER} --service ${ECS_SERVICE} --task-definition ${TASK_DEF} --region us-west-2'
   }
}
