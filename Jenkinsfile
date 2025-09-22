pipeline {

    agent {
        dockerfile {
            filename 'Dockerfile.ci'
            args '-u 0:0 --net host --privileged -v /var/run/docker.sock:/var/run/docker.sock'
        }
    }

    environment{
        
        GCR_CRED=credentials('gcp-func-service-account-key')
        GCP_PROJECT='activeproject-441912'
        PROJECT_NAME='mymyfunc'
        ENVVALUE='qa'
        TAG="${PROJECT_NAME}-${ENVVALUE}-${BUILD_NUMBER}"
    }

 stages {

    stage("Pipeline Metadata"){
        steps{
            script{
                try{
                    sh """
                        echo 'Build Number: ${BUILD_NUMBER}'
                        echo 'Git URL: ${GIT_URL}'
                        echo 'Git Branch: ${GIT_BRANCH}'
                        echo 'Git Commit: ${GIT_COMMIT}'
                        echo 'Build ID: ${BUILD_ID}'
                    """
                }
                catch(Exception e)
                {
                    echo "Pipeline metadata check failed: ${e.message}"
                    sh "exit 1"
                }       
            }
        }
    }

    stage('Install Dependencies'){
        steps{
            script{
                sh 'apk add --update --no-cache maven aws-cli jq'
            }
        }
    }

    stage('Zip Build') {
        steps {
            script{
                sh "zip -r ${TAG}.zip . -x 'Jenkinsfile' 'Dockerfile.ci' '*.git*' '*.vscode*'"
            }
        }  
    }

    stage('Upload to GCP') {
        steps {
            script {
				sh 'gcloud auth activate-service-account --key-file="$GCR_CRED"'
                sh 'gcloud config set project "${GCP_PROJECT}"'
                sh "gcloud storage ls"
                sh "gcloud storage cp ${TAG}.zip gs://run-sources-activeproject-441912-us-central1/services/myfunc/"
            }
        }
    }

    stage('Deploy to Cloud function') {
        steps {
           script{
                sh """gcloud functions deploy myfunc \
                       --gen2 \
                       --region=us-central1 \
                       --runtime=python311 \
                       --source=gs://run-sources-activeproject-441912-us-central1/services/myfunc/${TAG}.zip \
                       --entry-point=hello_http \
                       --trigger-http
                """
           }
        }
     }  
 }
    post { 
        always {
            cleanWs()
        }
    }
}


