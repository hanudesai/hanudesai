@Library("shared-lib") _

pipeline {
    agent any
    parameters {
        string defaultValue: 'token', name: 'auth', description: 'google auth'
        string defaultValue: 'apiName', name: 'apiName', description: 'ApiProxy Name'
        // choice choices: ['--PLEASE SELECT AN API--','ECCHUB_App_Token_v1','ECCHUB_CIAM_UserSession_v1'], name: 'API', description: 'Desired API to deploy/update on apigee.'
        choice choices: ['--PLEASE SELECT AN ENV--','DEV', 'UAT'], name: 'DESTINATION', description: 'Destination environment to deploy the apiproxy.'
        booleanParam defaultValue: false, name: 'deploy_target_server', description: 'Enables Target Servers Deployment'
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
        // stage('Authenticate') {
        //     when {
        //         expression { return params.dry_run == false }
        //     }
        //     steps {
        //         withCredentials([
        //             string(credentialsId: env.CREDENTIALS_APIGEE_SECRET_ID, variable: 'APIGEE_SECRET_ID'), 
        //             string(credentialsId: env.CREDENTIALS_APIGEE_CLIENT_ID, variable: 'APIGEE_CLIENT_ID')]) {
        
        //     script {
        //                 env.AUTH_KEY = sh(returnStdout: true, script: """curl --location -g --request POST '${env.URL_ENV}/apip/auth/v2/token' \\
        //                 --header 'Content-Type: application/json' \\
        //                 --data '{
        //                     "client_id": "$APIGEE_CLIENT_ID",
        //                     "client_secret": "$APIGEE_SECRET_ID",
        //                     "grant_type": "client_credentials"
        //                 }' | jq '.access_token' --raw-output """).trim()
        //                 sh """
        //                     set +x
        //                     RESULT=${env.AUTH_KEY}
        //                     if [[ "\$RESULT" == "null" ]]; then
        //                         echo "Error on Authentication"
        //                         exit 1;
        //                     fi
        //                     set -x
        //                 """
        //             }
        //     echo env.AUTH_KEY
        //         }
        //     }
        // }
        // stage('Checkout API Proxy') {
        //     steps {
        //          sh """
        //             echo "checkout API"
        //          """
        //         }
        //     }
        
        stage('Deploy Target Servers') {
            when {
                expression { return params.dry_run == false && params.deploy_target_server == true}
            }
            steps {
                sh """
                    echo "Running GET Request for TS"
                    deployApigeeProxy.getApigeeOrg(auth) 
                    deployApigeeProxy.getTargetServer("${env.ORGANIZATION}" ,  "${env.APIGEE_ENV}" ,  "test-htttpbin" ,  "${auth}" )
                    
                """
            }
        }
        stage('Build and Package API Proxy') {
            steps {
                dir("apiProxy/${apiName}") {
                    sh """                        
                       echo "Build stage"
                       pwd 
                       ls -lrt 
                       zip -r ${apiName}.zip apiproxy
                    """
                }
            }
        }
        stage('Import Proxy') {
            when {
                expression { return params.dry_run == false && params.import_proxy == true }
            }
            steps {
                sh """
                   echo "Import proxy"
                """
            }
        }
        stage('Deploy API Proxy') {
            when {
                expression { return false }
            }
            steps {
                sh """
                   echo "Deploy Proxy"
                """ 
            }
        }
        stage('Deploy Product') {
            when {
                expression { return params.dry_run == false && params.deploy_product == true}
            }
            steps {
                sh """
                    echo "Deploy Product"           
                """
            }
        }
        stage('Deploy Custom Attributes') {
            when {
                expression { return params.dry_run == false && params.deploy_custom_attributes == true}
            }
            steps {
                script {
                    sh """
                        echo "Deploy Custom Attribute"
                    """
                }
            }
        }
    }
}

