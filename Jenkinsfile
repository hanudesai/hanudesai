@Library("shared-lib") _
import groovy.json.JsonSlurper
pipeline {
    agent any
   /*
    parameters {
        string defaultValue: 'token', name: 'auth', description: 'google auth'
        string defaultValue: 'apiName', name: 'apiName', description: 'ApiProxy Name'
        choice choices: ['--PLEASE SELECT AN ENV--','DEV', 'UAT', 'PROD'], name: 'DESTINATION', description: 'Destination environment to deploy the apiproxy.'
        booleanParam defaultValue: false, name: 'override_target_server', description: 'override existing Target Servers Deployment'
        booleanParam defaultValue: false, name: 'deploy_proxy', description: 'Enables the API Proxy Import the code into apigee'
        booleanParam defaultValue: false, name: 'deploy_product', description: 'Enables Product Deployment'
    }
    environment {
        URIREPO = 'repolocation' // Repository proxies subgroup location
        HOST = 'github.com' // GH Hostname
        API_USER='user@email.com' // User email for api-user when call the cURL command.
    } 
    */
    stages {
                   stage('Parameters'){
                steps {
                    script {
                    properties([
                            parameters([
                                [$class: 'ChoiceParameter', 
                                    choiceType: 'PT_SINGLE_SELECT', 
                                    description: 'Select the Platform from the Dropdown List', 
                                    filterLength: 1, 
                                    filterable: false, 
                                    name: 'Platform', 
                                    script: [
                                        $class: 'GroovyScript', 
                                        fallbackScript: [
                                            classpath: [], 
                                            sandbox: false, 
                                            script: 
                                                "return['Could not get The platform list']"
                                        ], 
                                        script: [
                                            classpath: [], 
                                            sandbox: false, 
                                            script: 
                                                "return['Apigee-X','Apigee-AWS','Apigee-AKS']"
                                        ]
                                    ]
                                ],
                                [$class: 'CascadeChoiceParameter', 
                                    choiceType: 'PT_SINGLE_SELECT', 
                                    description: 'Select the Environment from the Dropdown List',
                                    name: 'Env', 
                                    referencedParameters: 'Platform', 
                                    script: 
                                        [$class: 'GroovyScript', 
                                        fallbackScript: [
                                                classpath: [], 
                                                sandbox: false, 
                                                script: "return['Could not get Environment from Env Param']"
                                                ], 
                                        script: [
                                                classpath: [], 
                                                sandbox: false, 
                                                script: '''
                                                if (Platform.equals("Apigee-X")){
                                                    return["Apigee-Dev", "Apigee-Test", "Apigee-Stage"]
                                                }
                                                else if(Platform.equals("Apigee-AWS")){
                                                    return["AWS-Dev", "AWS-Test", "AWS-Stage"]
                                                }
                                                else if(Platform.equals("Apigee-AKS")){
                                                    return["AKS-Dev", "AKS-Test", "AKS-Stage"]
                                                }
                                                '''
                                            ] 
                                    ]
                                ],
                                string(defaultValue: 'token', name: 'auth', description: 'google auth'),
                                string( defaultValue: 'apiName', name: 'apiName', description: 'ApiProxy Name'),
      //  choice choices: ['--PLEASE SELECT AN ENV--','DEV', 'UAT', 'PROD'], name: 'DESTINATION', description: 'Destination environment to deploy the apiproxy.'
                                booleanParam (defaultValue: false, name: 'override_target_server', description: 'override existing Target Servers Deployment'),
                                booleanParam (defaultValue: false, name: 'deploy_proxy', description: 'Enables the API Proxy Import the code into apigee'),
                                booleanParam (defaultValue: false, name: 'deploy_product', description: 'Enables Product Deployment')
                            ])
                        ])
                    }
                }
            }
        stage('Parse Environment Vars') {
            steps {
                script {
                    switch(params.Env){
                        case "Apigee-Dev":
                            env.URL_ENV = "https://apigee.googleapis.com";
                            env.CREDENTIALS_APIGEE_SECRET_ID = "APIGEE_DEV_SECRET_ID";
                            env.CREDENTIALS_APIGEE_CLIENT_ID = "APIGEE_DEV_CLIENT_ID";
                            env.ORGANIZATION = "cps-apg-hybrid";
                            env.projectid = "cps-apg-hybrid"
                            env.APIGEE_ENV = "default-dev"
                            // env.WHICHBRANCH = "";
                            break;
                        case "Apigee-Test":
                            env.URL_ENV = "https://apigee.googleapis.com";
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
                expression { return params.Env == "Apigee-Dev" }
            }
            steps {
                sh """
                    echo "Deploy TargetServer  to DEV ENV" 
                """
                script{
                   if (override_target_server == "true") {
                    deployApigeeProxy.deployTargetServer(org:"${env.ORGANIZATION}" ,  env:"${env.APIGEE_ENV}" ,  targetServer: "test-htttpbin" ,  auth: "${auth}" , targetOverride: "${params.override_target_server}")
                    }
                } 
                dir("API/${apiName}") {
                    script {
                    if (deploy_proxy == "true" ){
                        sh " apigeelint -s apiproxy/ -f table.js "
                        sh """                        
                            echo "Build stage"
                            pwd 
                            ls -lrt 
                            apigeelint -s apiproxy -f table.js 
                            zip -r ${apiName}.zip apiproxy
                        """
                            echo "deploy API Proxy"
                            def zipFile = "${apiName}.zip"
                            def (val_response , val_null) = apigeePost.post("${auth}" , "${env.ORGANIZATION}" , "$apiName" , "validate" , "${zipFile}" )
                            println val_response
                            if(val_response.status == 200 )  {
                                def ( import_response , new_api_revision )  = apigeePost.post("${auth}" , "${env.ORGANIZATION}" , "$apiName" , "import" , "${zipFile}" )
                                println import_response.status 
                                println new_api_revision
                                def deploy_api = apigeePost.updateRevision( "${auth}" , "${env.ORGANIZATION}" , "$apiName" , "$new_api_revision" , "$env.APIGEE_ENV" )
                            }
                        }
                    }
                }
                script{
                     echo "Deploy Product"
                     println  deploy_product
                    if (deploy_product == "true" ) {
                         echo "Deploy Product"
                        deployApigeeProducts.deployProducts(org:"${env.ORGANIZATION}"  ,  auth: "${auth}" , apiName : "${apiName}" )
                    }
                }
            }
        }
        stage('Deploy API to TEST ENV') {
            when {
                expression { return params.Env == "TEST" }
            }
            steps {
                sh """
                    echo "Deploy TargetServer  to DEV ENV" 
                """
                script{
                    deployApigeeProxy.deployTargetServer(org:"${env.ORGANIZATION}" ,  env:"${env.APIGEE_ENV}" ,  targetServer: "test-htttpbin" ,  auth: "${auth}" , targetOverride: "${params.override_target_server}")
                }
                dir("API/${apiName}") {
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
        stage('Deploy API to Stage ENV') {
            when {
                expression { return params.Env == "PROD" }
            }
            steps {
                sh """
                    echo "Deploy TargetServer  to PROD ENV" 
                """
                script{
                    deployApigeeProxy.deployTargetServer(org:"${env.ORGANIZATION}" ,  env:"${env.APIGEE_ENV}" ,  targetServer: "test-htttpbin" ,  auth: "${auth}" , targetOverride: "${params.override_target_server}")
                }
                dir("API/${apiName}") {
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