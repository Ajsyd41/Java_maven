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
        FUNCTION_NAME='mydepfunc'
        REGION='us-central1'
        RUNTIME='python311'
        TIMEOUT='120s'
        DEPLOYMENT_FOLDER='functiondeployfolder'
        ENTRYPOINT='hello_http'
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

    stage('Authenticate to GCP') {
        steps {
            script {
				sh 'gcloud auth activate-service-account --key-file="$GCR_CRED"'
                sh 'gcloud config set project "${GCP_PROJECT}"'
            }
        }
    }

    stage('Move files') {
        steps {
            script {
                sh '''
                    mkdir "${DEPLOYMENT_FOLDER}"
                    mv main.py requirements.txt ./"${DEPLOYMENT_FOLDER}"
                    ls -la ./"${DEPLOYMENT_FOLDER}"
                '''
            }
        }
    }

    stage('Deploy Cloud function') {
        steps {
           script{
                sh """gcloud functions deploy ${FUNCTION_NAME} \
                       --gen2 \
                       --region=${REGION} \
                       --runtime=${RUNTIME} \
                       --timeout=${TIMEOUT} \
                       --source=./${DEPLOYMENT_FOLDER} \
                       --entry-point=${ENTRYPOINT} \
                       --allow-unauthenticated \
                       --trigger-http
                """
           }
        }
     }

    stage('Remove files') {
        steps {
            script {
                sh 'rm -rf ./"${DEPLOYMENT_FOLDER}"'
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


