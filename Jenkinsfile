@Library("shared-lib") _

pipeline {
    agent any
    parameters {
        string defaultValue: 'token', name: 'auth', description: 'google auth'
        string defaultValue: 'apiName', name: 'apiName', description: 'ApiProxy Name'
        // choice choices: ['--PLEASE SELECT AN API--','ECCHUB_App_Token_v1','ECCHUB_CIAM_UserSession_v1'], name: 'API', description: 'Desired API to deploy/update on apigee.'
        choice choices: ['--PLEASE SELECT AN ENV--','DEV', 'UAT', 'PROD'], name: 'DESTINATION', description: 'Destination environment to deploy the apiproxy.'
        booleanParam defaultValue: false, name: 'override_target_server', description: 'override existing Target Servers Deployment'
        booleanParam defaultValue: false, name: 'import_proxy', description: 'Enables the API Proxy Import the code into apigee'
        booleanParam defaultValue: false, name: 'deploy_product', description: 'Enables Product Deployment'
        booleanParam defaultValue: false, name: 'deploy_custom_attributes', description: 'Enables Custom Attributes Deployment'
        booleanParam defaultValue: true, name: 'dry_run', description: 'Just make a dry-run without deploy anything'
    }
    environment {
        URIREPO = 'repolocation' // Repository proxies subgroup location
        HOST = 'github.com' // GH Hostname
        API_USER='user@email.com' // User email for api-user when call the cURL command.
    }
    stages {
       

        stage('Parse Environment Vars') {
            steps {
                script {
                    switch(params.DESTINATION){
                        case "DEV":
                            env.URL_ENV = "https://apigee_Url";
                            env.CREDENTIALS_APIGEE_SECRET_ID = "APIGEE_DEV_SECRET_ID";
                            env.CREDENTIALS_APIGEE_CLIENT_ID = "APIGEE_DEV_CLIENT_ID";
                            env.ORGANIZATION = "cps-apg-hybrid";
                            env.projectid = "cps-apg-hybrid"
                            env.APIGEE_ENV = "default-dev"
                            // env.WHICHBRANCH = "";
                            break;
                        case "UAT":
                            env.URL_ENV = "https://apigee_Url";
                            env.CREDENTIALS_APIGEE_SECRET_ID = "APIGEE_QA_SECRET_ID";
                            env.CREDENTIALS_APIGEE_CLIENT_ID = "APIGEE_QA_CLIENT_ID";
                            env.ORGANIZATION = "qa";
                            env.WHICHBRANCH = "dev";
                            break;

                    }
                }
                echo env.URL_ENV
            }
        }
     
        
        stage('Deploy API to DEV ENV') {
            when {
                expression { return params.DESTINATION == "DEV" }
            }
            steps {
                sh """
                    echo "Deploy TargetServer  to DEV ENV" 
                """
                script{
                   
                    deployApigeeProxy.deployTargetServer(org:"${env.ORGANIZATION}" ,  env:"${env.APIGEE_ENV}" ,  targetServer: "test-htttpbin" ,  auth: "${auth}" , targetOverride: "${params.override_target_server}")
                    
                
                }
                dir("apiProxy/${apiName}") {
                    sh """                        
                       echo "Build stage"
                       pwd 
                       ls -lrt 
                       zip -r ${apiName}.zip apiproxy
                    """
                    script {
                        echo "deploy API Proxy"
                        deployApigeeProxy.deployApiProxy("${auth}" , "${env.ORGANIZATION}" , "$apiName")
                        
                    }
                }
                
            }
        }

                stage('Deploy API to UAT ENV') {
            when {
                expression { return params.DESTINATION == "UAT" }
            }
            steps {
                sh """
                    echo "Deploy TargetServer  to DEV ENV" 
                """
                script{
                   
                    deployApigeeProxy.deployTargetServer(org:"${env.ORGANIZATION}" ,  env:"${env.APIGEE_ENV}" ,  targetServer: "test-htttpbin" ,  auth: "${auth}" , targetOverride: "${params.override_target_server}")
                    
                
                }
                dir("apiProxy/${apiName}") {
                    sh """                        
                       echo "Build stage"
                       pwd 
                       ls -lrt 
                       zip -r ${apiName}.zip apiproxy
                    """
                    script {
                        echo "deploy API Proxy"
                        deployApigeeProxy.deployApiProxy("${auth}" , "${env.ORGANIZATION}" , "$apiName")
                        
                    }
                }
                
            }
        }
    
                    stage('Deploy API to PROD ENV') {
            when {
                expression { return params.DESTINATION == "PROD" }
            }
            steps {
                sh """
                    echo "Deploy TargetServer  to PROD ENV" 
                """
                script{
                   
                    deployApigeeProxy.deployTargetServer(org:"${env.ORGANIZATION}" ,  env:"${env.APIGEE_ENV}" ,  targetServer: "test-htttpbin" ,  auth: "${auth}" , targetOverride: "${params.override_target_server}")
                    
                
                }
                dir("apiProxy/${apiName}") {
                    sh """                        
                       echo "Build stage"
                       pwd 
                       ls -lrt 
                       zip -r ${apiName}.zip apiproxy
                    """
                    script {
                        echo "deploy API Proxy"
                        
                    }
                }
                
            }
        }
       

    }
}

