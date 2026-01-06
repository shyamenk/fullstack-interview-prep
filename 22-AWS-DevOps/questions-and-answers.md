# AWS DevOps Interview Questions & Answers

## Table of Contents

1. [CodePipeline vs Jenkins](#q1-how-does-codepipeline-compare-to-jenkins)
2. [CloudFormation vs Terraform](#q2-what-is-the-difference-between-cloudformation-and-terraform)
3. [Blue/Green vs Canary Deployment](#q3-what-is-the-difference-between-bluegreen-and-canary-deployment)
4. [Infrastructure as Code (IaC) Benefits](#q4-why-use-infrastructure-as-code)
5. [CodeBuild vs CodeDeploy](#q5-what-is-the-difference-between-codebuild-and-codedeploy)
6. [Elastic Beanstalk](#q6-what-is-elastic-beanstalk-and-when-to-use-it)
7. [CloudFormation Drift](#q7-what-is-cloudformation-drift-detection)
8. [AWS CDK (Cloud Development Kit)](#q8-what-is-aws-cdk-and-how-is-it-different-from-cloudformation)
9. [Systems Manager (SSM) Parameter Store](#q9-what-is-ssm-parameter-store)
10. [Secrets Manager vs Parameter Store](#q10-when-should-you-use-secrets-manager-vs-parameter-store)
11. [OpsWorks](#q11-what-is-aws-opsworks)
12. [CloudWatch Logs Insights](#q12-what-is-cloudwatch-logs-insights)
13. [Immutable Infrastructure](#q13-what-is-immutable-infrastructure)
14. [CodeCommit](#q14-what-is-codecommit)
15. [EC2 User Data vs AMI](#q15-what-is-the-trade-off-between-using-user-data-scripts-and-baking-amis)
16. [Cross-Region Replication (CodePipeline)](#q16-how-do-you-deploy-to-multiple-regions-using-codepipeline)
17. [CloudFormation Nested Stacks](#q17-what-are-nested-stacks-in-cloudformation)
18. [ECR (Elastic Container Registry)](#q18-what-is-ecr)
19. [Run Command (SSM)](#q19-how-do-you-patch-100-ec2-instances-simultaneously)
20. [RTO vs RPO](#q20-explain-rto-and-rpo-in-disaster-recovery)

---

## Q1. How does CodePipeline compare to Jenkins?
Concept:
- **CodePipeline:** Fully managed, serverless, integrates natively with AWS (IAM, S3, CodeDeploy). No plugins to manage. Pay per active pipeline.
- **Jenkins:** Open source, self-hosted (EC2). Huge plugin ecosystem. requires maintenance (patching, scaling). Free software but costs compute time.

Analogy:
- **CodePipeline:** Hiring a moving company (AWS) to move your boxes.
- **Jenkins:** Buying your own truck to move boxes. You have to drive, gas up, and fix the engine.

⚡ One-liner to remember:
CodePipeline = Managed AWS CI/CD; Jenkins = Flexible but self-managed Server.

## Q2. What is the difference between CloudFormation and Terraform?
Concept:
- **CloudFormation:** AWS Native (JSON/YAML). State is managed by AWS. Deep integration with AWS specific features.
- **Terraform:** Open Source (HCL). Multi-cloud (AWS, Azure, GCP). State file managed locally or in S3.

Analogy:
- **CloudFormation:** Speaking the local dialect (AWS Native).
- **Terraform:** Using a universal translator (works everywhere).

⚡ One-liner to remember:
CloudFormation for pure AWS; Terraform for multi-cloud strategies.

## Q3. What is the difference between Blue/Green and Canary Deployment?
Concept:
- **Blue/Green:** Two identical environments. Switch traffic 100% from Old (Blue) to New (Green) instantly. Fast rollback. High cost (duplicate resources).
- **Canary:** Slowly shift traffic (10% -> 50% -> 100%) to the new version. Validation at each step. Lower risk, longer time.

Analogy:
- **Blue/Green:** Flipping a light switch. Instant change.
- **Canary:** Testing the water with your toe before jumping in.

⚡ One-liner to remember:
Blue/Green = Instant switch (Costly); Canary = Gradual rollout (Safe).

## Q4. Why use Infrastructure as Code (IaC)?
Concept:
Managing infrastructure using code/templates (Git) rather than manual console clicks.
- **Consistency:** No configuration drift.
- **Speed:** Automated provisioning.
- **Version Control:** History of changes.
- **Reusability:** Deploy same stack to Dev, Test, Prod.

⚡ One-liner to remember:
Treat your servers like software (Versioned, Reproducible, Automated).

## Q5. What is the difference between CodeBuild and CodeDeploy?
Concept:
- **CodeBuild (CI):** Compiles code, runs tests, produces artifacts (WAR/ZIP/Docker Image).
- **CodeDeploy (CD):** Takes the artifact and installs it onto targets (EC2, Lambda, ECS).

Analogy:
- **CodeBuild:** The factory that assembles the car.
- **CodeDeploy:** The truck that delivers the car to the dealership.

⚡ One-liner to remember:
Build creates the package; Deploy installs the package.

## Q6. What is Elastic Beanstalk and when to use it?
Concept:
PaaS offering. Handles capacity provisioning, load balancing, scaling, and health monitoring. You just upload code.
Use for simple web apps when you don't want to manage infrastructure but want more control than Heroku.

Analogy:
A turnkey apartment. Furniture and utilities are set up; you just bring your clothes (Code).

⚡ One-liner to remember:
Easiest way to deploy web apps on AWS without configuring servers manually.

## Q7. What is CloudFormation Drift Detection?
Concept:
A feature that checks if the actual stack resources have been modified manually (via Console/CLI) outside of the CloudFormation template.
Helps detect configuration drift.

Analogy:
A building inspector checking if the house matches the original blueprints. "Hey, someone added a window here that wasn't in the plan!"

⚡ One-liner to remember:
Detects manual changes that deviate from the IaC template.

## Q8. What is AWS CDK and how is it different from CloudFormation?
Concept:
**Cloud Development Kit (CDK)** allows defining infrastructure using **real programming languages** (TypeScript, Python, Java) instead of JSON/YAML.
It *compiles down* to CloudFormation templates.

Analogy:
- **CloudFormation:** Writing Assembly code (Raw, verbose).
- **CDK:** Writing Python (High-level, loops, conditionals, abstraction).

⚡ One-liner to remember:
Infrastructure as *actual* Code (imperative), not just Configuration (declarative).

## Q9. What is SSM Parameter Store?
Concept:
Secure, hierarchical storage for configuration data management and secrets management.
- Stores strings, lists, and secure strings (Encrypted).
- Free (Standard tier).
- Integrated with IAM.

⚡ One-liner to remember:
Store config vars (DB URLs, API Keys) centrally and inject them into apps.

## Q10. When should you use Secrets Manager vs Parameter Store?
Concept:
- **Parameter Store:** Free, simple storage for config/secrets.
- **Secrets Manager:** Paid service. Designed specifically for DB credentials. Supports **automatic rotation** of keys (e.g., change RDS password every 30 days).

⚡ One-liner to remember:
Use Secrets Manager if you need automatic key rotation; otherwise, Parameter Store is cheaper.

## Q11. What is AWS OpsWorks?
Concept:
Configuration management service that uses **Chef** and **Puppet**. Automates how servers are configured, deployed, and managed.
Mostly legacy/specific use cases now; Systems Manager is preferred.

⚡ One-liner to remember:
Managed Chef/Puppet for AWS.

## Q12. What is CloudWatch Logs Insights?
Concept:
Interactive query capability for CloudWatch Logs.
Allows you to write SQL-like queries to aggregate, filter, and visualize log data instantly.
e.g., "Show me the 20 most recent errors".

⚡ One-liner to remember:
SQL-like querying tool for your log files.

## Q13. What is Immutable Infrastructure?
Concept:
Once a server is deployed, it is **never modified**. To update, you replace the entire server with a new one.
- Prevents configuration drift.
- Simplifies rollback (just switch back to old image).

Analogy:
Disposable plates. You don't wash them (Update); you throw them away and get a new one.

⚡ One-liner to remember:
Don't patch servers; replace them.

## Q14. What is CodeCommit?
Concept:
Managed Git source control service. Similar to GitHub/GitLab but hosted on AWS.
Secure, highly scalable, integrated with IAM.

⚡ One-liner to remember:
AWS's version of private GitHub repositories.

## Q15. What is the trade-off between using User Data scripts and baking AMIs?
Concept:
- **User Data:** Script runs on boot (e.g., `yum install apache`).
    - *Pro:* Flexible, easy to change.
    - *Con:* Slow boot time (installs happen every launch).
- **Golden AMI:** Pre-install everything and save image.
    - *Pro:* Fast boot time.
    - *Con:* Hard to manage (must bake new AMI for every small change).

⚡ One-liner to remember:
Bake AMIs for speed (Immutable); use User Data for flexibility.

## Q16. How do you deploy to multiple regions using CodePipeline?
Concept:
CodePipeline supports cross-region actions. You can add a "Deploy" stage that targets a different region (e.g., Deploy to us-east-1, then wait for approval, then Deploy to eu-west-1).
Requires passing artifacts via S3 buckets in respective regions.

⚡ One-liner to remember:
Add multiple Deploy stages in the pipeline targeting different regions.

## Q17. What are Nested Stacks in CloudFormation?
Concept:
Calling one CloudFormation template from within another.
Allows breaking huge templates into smaller, reusable components (e.g., Networking Stack, Database Stack, App Stack).

Analogy:
Functions in programming. Instead of one 10,000 line `main()`, you break it into functions (`networking()`, `database()`).

⚡ One-liner to remember:
Modularize your IaC by creating stacks within stacks.

## Q18. What is ECR?
Concept:
**Elastic Container Registry.** A fully managed Docker container registry.
Where you store your Docker images to be used by ECS, EKS, or Lambda.

⚡ One-liner to remember:
AWS's version of Docker Hub.

## Q19. How do you patch 100 EC2 instances simultaneously?
Concept:
Use **AWS Systems Manager (SSM) Run Command** or **Patch Manager**.
It executes commands on fleets of instances without SSH-ing into them individually.

⚡ One-liner to remember:
Use SSM Run Command to execute scripts at scale without SSH.

## Q20. Explain RTO and RPO in Disaster Recovery.
Concept:
- **RTO (Recovery Time Objective):** Max acceptable downtime. "How long can we be offline?" (e.g., 1 hour).
- **RPO (Recovery Point Objective):** Max acceptable data loss. "How much data can we lose?" (e.g., 5 mins).

Analogy:
- **RTO:** How fast the ambulance arrives.
- **RPO:** How much memory you lost from the accident.

⚡ One-liner to remember:
RTO = Time to restore; RPO = Data loss tolerance.
