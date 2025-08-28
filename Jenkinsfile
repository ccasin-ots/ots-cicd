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

        stage('Regression Test') {
            steps {
                sh '''#!/bin/bash
                set -e

                echo "Running Postman regression tests..."

                # Run Newman tests
                newman run regressiontest/RegressionTest.postman_collection.json \
                  --reporters cli,junit \
                  --reporter-junit-export results.xml
                '''
            }
        }
    }
}
