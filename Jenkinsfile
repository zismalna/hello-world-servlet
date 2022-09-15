/* properties([
  parameters([
    string(name: 'submodule', defaultValue: ''),
    string(name: 'submodule_branch', defaultValue: ''),
    string(name: 'commit_sha', defaultValue: ''),
  ])
])
def artifact_bucket = 'default'
 */



pipeline{
    agent none
    tools{
        terraform 'terraform'
    }
    stages{
        stage("build"){
            agent{
                docker {
                image 'maven:3.8.1-adoptopenjdk-11'
                args '-v /root/.m2:/root/.m2'
                }
            }
            steps{
                echo "========Building package========"
                sh 'mvn -B clean package'
                stash includes: 'target/helloworld.war', name: 'builded_war'
            }
            post{
                always{
                    echo "========always========"
                }
                success{
                echo "========Build executed successfully========"
                }
                failure{
                    echo "========A execution failed========"
                }
            }

        }
        stage("deploy"){
            agent any
            steps{
                git branch: 'main', url: 'https://git.epam.com/mykhailo_lopaiev/pet-project-aws-and-ci-cd', credentialsId: 'github-deploy'
                echo "========Uploading to s3========"
                echo "copying to s3"

                withAWS(credentials:'awsAKIA6JPPVFU3AX6YK47O', region: 'eu-central-1'){
                sh 'pwd'
                dir ('Terraform/environments/dev'){
                sh 'terraform workspace new dev'
                sh 'terraform init'
                sh 'terraform plan -out tfplan'
                sh 'terraform apply tfplan'
                }
                script{
                def artifact_bucket = sh(returnStdout: true, script: 'terraform output -raw data_bucket_name')
                }
                unstash 'builded_war'
                s3Upload(file:'helloworld.war', bucket:"${artifact_bucket}", path:'build/helloworld.war')
                }
            }
        }

    }
    post{
        always{
            echo "========always========"
            
        }
        success{
            echo "========pipeline executed successfully ========"
        }
        failure{
            echo "========pipeline execution failed========"
        }
    }
}