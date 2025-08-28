pipeline {
    agent any
    options {
        buildDiscarder(logRotator(numToKeepStr: '10', artifactNumToKeepStr: '1'))
        disableConcurrentBuilds()
    }

    stages {
        stage('Checkout') {
            steps {
                echo "Repo checked out successfully."
            }
        }

        stage('List files') {
            steps {
                sh 'ls -R'
            }
        }

        stage('Deploy') {
            steps {
                dir('repositories') {
                    sh '''#!/bin/bash
                        set -e
                        
                        TOKEN=$(curl -s -k --location \
                          'https://ec2-18-140-203-30.ap-southeast-1.compute.amazonaws.com/ots/keycloak/realms/PAS/protocol/openid-connect/token' \
                          --header 'Content-Type: application/x-www-form-urlencoded' \
                          --data-urlencode 'username=ccasin' \
                          --data-urlencode 'password=Asdf!234' \
                          --data-urlencode 'client_id=bridge' \
                          --data-urlencode 'grant_type=password' \
                          --data-urlencode 'client_secret=password' \
                        | jq -r '.access_token')
                        
                        echo "Got token: $(echo $TOKEN | cut -c1-20)..."
                        
                        curl -k -X POST \
                          'https://ec2-18-140-203-30.ap-southeast-1.compute.amazonaws.com/ots/bridge/bridge/rest/services?overwrite=true&overwritePrefs=false&startup=true&preserveNodeModules=false&npmInstall=false&runScripts=false&stopTimeout=10&allowKill=false' \
                          -H "Authorization: Bearer $TOKEN" \
                          -H 'accept: application/json' \
                          -H 'Content-Type: multipart/form-data' \
                          -F 'uploadFile=@randomTeamSelect.rep'
                        '''
                }
            }
        }

        stage('Run Test Command') {
            steps {
                // Example running your jar if needed
                sh 'java -jar tools/xumlc-7.20.0.jar -h || true'
            }
        }

        stage('Regression Test') {
            steps {
                sh '''#!/bin/bash
                set -e

                echo "Running regression test against RandomTeamGeneratorCICD API..."

                RESPONSE=$(curl -s -w "%{http_code}" --location \
                  'https://ec2-18-140-203-30.ap-southeast-1.compute.amazonaws.com/ots/gateway/BSPDemoPolicy/RandomTeamGeneratorCICD/1.0/randomize/TEAM?apikey=44093a95-c01c-4bb6-979d-fcd6bdaa888b' \
                  --header 'accept: application/json' \
                  --header 'X-API-Key: 44093a95-c01c-4bb6-979d-fcd6bdaa888b' \
                  --header 'Cookie: OAuth_Token_Request_State=7a7b1fd2-b32c-4b59-9428-3b0891559889; SESSIONID_20210531035415=5EE3149531BC048EC5A69DCD381A9102' \
                  -o output.json)

                STATUS_CODE=$(tail -n1 <<< "$RESPONSE")
                
                echo "HTTP Status: $STATUS_CODE"
                echo "Response body:"
                cat output.json

                if [ "$STATUS_CODE" -ne 200 ]; then
                    echo "Regression test failed!"
                    exit 1
                fi
                '''
            }
        }
    }
}
