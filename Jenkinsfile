pipeline {
    agent any
    environment { 
        PROJECT_NAME 					= 'WebApplication1'
        PROJECT_S3_BUCKET_REGION 		= 'ap-south-1'
        PROJECT_S3_BUCKET_NAME 			= 'sample-net-webapp'
        PROJECT_BUILD_OUTPUT_FILE_NAME    	= 'WebApplication1.zip'              
        PROJECT_SOLUTION_NAME 			= 'WebApplication1.sln'
    }
    stages {
        stage('Cloning the project repository from BitBucket') {
            steps {
                checkout(
                    [
                        $class: 'GitSCM', 
                        branches: [[name: '*/master']], 
                        doGenerateSubmoduleConfigurations: false, 
                        extensions: [[$class: 'CleanBeforeCheckout']], 
                        submoduleCfg: [], 
                        userRemoteConfigs: [
                            [
                                credentialsId: 'git', 
                                url: 'https://github.com/Kunreddi/WebApplication1.git'
                            ]
                        ]
                    ]
                )
            }
        }

        stage('Building the project') {
            steps {
                //bat 'nuget restore ${PROJECT_SOLUTION_NAME}'
		        bat "\"${tool 'Microsoft Visual Studio'}\\msbuild.exe\" ${PROJECT_SOLUTION_NAME} /p:DeployOnBuild=true /p:PublishProfile=FolderProfile /p:Configuration=Release /p:Platform=\"Any CPU\" /p:ProductVersion=1.0.0.${env.BUILD_NUMBER}"
            }
        }

        stage('Uploading application package to s3') {
            steps {
                withCredentials([[
                $class: 'AmazonWebServicesCredentialsBinding',
                accessKeyVariable: 'AWS_ACCESS_KEY_ID', // dev credentials
                credentialsId: 'AWSCRED',
                secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                ]]){
                    powershell '''
                        Import-Module AWSPowerShell
                        Set-AWSCredentials -AccessKey "$($ENV:AWS_ACCESS_KEY_ID)" -SecretKey "$($ENV:AWS_SECRET_ACCESS_KEY)" -StoreAs "$($ENV:PROJECT_NAME)"
                        Write-S3Object -BucketName "$($ENV:PROJECT_S3_BUCKET_NAME)" -File "$($ENV:PROJECT_BUILD_OUTPUT_FILE_NAME)" -Region "$($ENV:PROJECT_S3_BUCKET_REGION)" -ProfileName "$($ENV:PROJECT_NAME)"
                    '''
                }
            }
        }
    }
 }

