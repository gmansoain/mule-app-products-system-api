pipeline {
    agent any
    stages {
        stage('Create APIM instance') {
            steps {
                script{
                   sh '''
                        response=$(curl --location 'https://anypoint.mulesoft.com/accounts/login' \
                        --header 'Content-Type: application/x-www-form-urlencoded' \
                        --data-urlencode 'username=gonmule' \
                        --data-urlencode 'password=Scipio235ac!')
                        token=$(echo $response | jq '.access_token')
                        token2=$(echo "$token" | tr -d '"')
                        auth_token="Authorization: Bearer ${token2}"
                        echo $auth_token
                        auth_token2="'$auth_token'"
                        api=$(curl --location 'https://anypoint.mulesoft.com/apimanager/api/v1/organizations/fc9e3283-a489-4e88-99aa-cceea4aa5ea1/environments/95f97b54-58e2-4c1a-8f9b-c5cc0fed4291/apis' \
                                --header "$auth_token" \
                                --header 'Content-Type: application/json' \
                                --data '{
                                    "endpoint":{
                                        "deploymentType":"CH",
                                        "isCloudHub":null,
                                        "muleVersion4OrAbove": true,
                                        "proxyUri":null,
                                        "referencesUserDomain":null,
                                        "responseTimeout":null,
                                        "type":"raml",
                                        "uri":null
                                    },
                                    "providerId":null,
                                    "instanceLabel":null,
                                    "spec":{
                                        "assetId":"products-system-api",
                                        "groupId":"fc9e3283-a489-4e88-99aa-cceea4aa5ea1",
                                        "version":"1.0.0"
                                    }
                                }'
                            )
                        echo $api
                        apim=$(echo $api | jq '.id')
                        echo $apim 
                        echo $apim > ~/apim_file
                   '''
                }
            }
        }
    
        stage('Check Out') {
            steps {
                git 'https://github.com/gmansoain/mule-app-products-system-api.git'
            }
        }
        
        stage('Build and Publish to Exchange'){
            steps{
                sh 'cd /var/jenkins_home/workspace/PRODUCTS-SYSTEM-API/products-system-api && mvn clean deploy'
            }
        }
        
        stage('Deploy to CH2.0'){
            steps{
                sh '''
                    apim=$(cat ~/apim_file)
                    echo $apim
                    cd /var/jenkins_home/workspace/PRODUCTS-SYSTEM-API/products-system-api
                    mvn deploy -DmuleDeploy -Dapi.id=$apim -Denv=prod -Dapi.layer=System -Dsecure.key=Mule1234 -Dusername=gonmule -Dpassword=Scipio235ac! -Dclient_id=6cf41b1f668642ed971f4c9268e440b9 -Dclient_secret=5f10d50bc9EC44F08F51b736Ec837d3D
                '''
            }
        }
        stage('Readiness Tests'){
            steps{
                sh '''
                	cd /var/jenkins_home/workspace/PRODUCTS-SYSTEM-API/products-system-api/postman
                	newman run ./products-system-api-testing.json
                '''
            }
        }
    }
}
