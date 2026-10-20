Permission for codebuild role :


Hands-On Lab: CI/CD Pipeline for Spring Boot on Amazon ECS
Architecture Overview
    1. GitHub Repository: Stores source code & buildspec.yml.
       
    2. AWS CodePipeline: Detects pushes from GitHub.
       
    3. AWS CodeBuild: Compiles Java app, builds Docker image, pushes to ECR, and registers/updates ECS.
       
    4. Amazon ECS (Fargate): Runs the containerized application serverless-style.
       
Step 1: Prepare the ECS Infrastructure (The Foundation)
Before building the pipeline, ECS needs a target to deploy to.

    1. Create an ECR Repository:
       
        ◦ Go to Amazon ECR -> Repositories -> Create repository.
          
        ◦ Name: spring-demo. Click Create.
          
    2. Create an ECS Cluster:
       
        ◦ Go to Amazon ECS -> Clusters -> Create cluster.
          
        ◦ Name: spring-ecs-cluster.
          
        ◦ Infrastructure: Choose AWS Fargate (serverless). Click Create.
          
    3. Create an Initial Task Definition (Blueprint):
       
        ◦ Go to Task Definitions -> Create new task definition.
          
        ◦ Family name: spring-task.
          
        ◦ Launch type: AWS Fargate.
          
        ◦ Container details: Name it spring-app, and for Image URI, use a temporary placeholder like nginx:latest (Port 8080). Click Create.
          
    4. Create an ECS Service:
       
        ◦ Open your spring-ecs-cluster -> Services tab -> Create.
          
        ◦ Launch type: Fargate. Task definition: spring-task.
          
        ◦ Service name: mytask-service. Desired tasks: 1. Click Create.
          
Step 2: Configure IAM Permissions for CodeBuild
CodeBuild needs permissions to push to ECR, describe tasks, register task definitions, and update ECS services.

    1. Go to the IAM Console -> Roles.
       
    2. Find your CodeBuild role (typically named codebuild-<project-name>-service-role).
       
    3. Click Add permissions -> Create inline policy -> Switch to the JSON tab and paste:
       
JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ECRPermissions",
            "Effect": "Allow",
            "Action": [
                "ecr:GetAuthorizationToken",
                "ecr:BatchCheckLayerAvailability",
                "ecr:CompleteLayerUpload",
                "ecr:UploadLayerPart",
                "ecr:InitiateLayerUpload",
                "ecr:PutImage",
                "ecr:DescribeRepositories",
                "ecr:CreateRepository"
            ],
            "Resource": "*"
        },
        {
            "Sid": "ECSPermissions",
            "Effect": "Allow",
            "Action": [
                "ecs:UpdateService",
                "ecs:DescribeServices",
                "ecs:DescribeTaskDefinition",
                "ecs:RegisterTaskDefinition"
            ],
            "Resource": "*"
        },
        {
            "Sid": "IAMPASSROlePermission",
            "Effect": "Allow",
            "Action": [
                "iam:PassRole"
            ],
            "Resource": "*",
            "Condition": {
                "StringLike": {
                    "iam:PassedToService": "ecs-tasks.amazonaws.com"
                }
            }
        }
    ]
}
    4. Name the policy CodeBuildECSECSDeploymentPolicy and click Create policy.
       
Step 3: Set up the GitHub Repository & buildspec.yml
In the root directory of your project repository on GitHub, ensure you have your Dockerfile inside your app folder (e.g., hello-gkc/Dockerfile) and your buildspec.yml at the absolute root.

Find buildspec.yml in git:

Step 4: Create the CodePipeline
    1. Open the CodePipeline Console -> Create pipeline. 
       
    2. Name your pipeline and choose New service role. 
       
    3. Source Stage: Select GitHub (via GitHub App), authenticate, and point to your repository and branch.
       
    4. Build Stage: Select AWS CodeBuild -> Click Create project: 
       
        ◦ Project name: spring-build-project.
          
        ◦ Environment: Managed image -> Ubuntu -> Standard runtime. 
          
        ◦ Buildspec: Choose Use buildspec file.
          
        ◦ Complete project creation and return to the wizard.
          
    5. Deploy Stage: Click Skip deploy stage (since CodeBuild handles the ECS deployment natively via the buildspec script).
       
    6. Review settings and click Create pipeline. 
       
Step 5: Test and Verify
    1. Commit and push your code updates to GitHub:
       
       Bash
       git add .
       git commit -m "Ready for CI/CD deployment lab"
       git push origin main
    2. Watch CodePipeline automatically pick up the code change, compile it via Maven, build the image, push it to ECR, and execute the ECS rolling update.
       
 an additional walkthrough of connecting GitHub repository events to CodeBuild and deploying containers directly onto AWS ECS Fargate targets
