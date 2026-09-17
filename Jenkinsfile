// WORKING

pipeline {
    agent {
        docker {
            image 'cimg/node:22.17.0'
            args '-u root'
        }
    }

    environment {
        CORTEX_API_KEY = credentials('CORTEX_API_KEY')
        CORTEX_API_KEY_ID = credentials('CORTEX_API_KEY_ID')
        CORTEX_API_URL = 'https://api-japac-ccr.xdr.jp.paloaltonetworks.com'
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                // This is the step that stashes your files.
                // It is a crucial prerequisite for the 'unstash' command.
                checkout scm
                stash includes: '**/*', name: 'source'
            }
        }
        
        stage('Install Dependencies') {
            steps {
                sh '''
                apt update
                apt install -y curl jq git
                '''
            }
        }

        stage('Download Cortex CLI') {
    environment {
        CORTEX_API_KEY_ID = credentials('cortex-api-key-id')
        CORTEX_API_KEY    = credentials('cortex-api-key')
    }
    steps {
        sh '''
          set -e
          
          # Query download link safely using environment variables
          RESPONSE=$(curl -s --location "https://api-japac-ccr.xdr.jp.paloaltonetworks.com/public_api/v1/unified-cli/releases/download-link?os=linux&architecture=amd64" \
            -H "x-xdr-auth-id: ${CORTEX_API_KEY_ID}" \
            -H "Authorization: ${CORTEX_API_KEY}")

          # Extract URL
          DOWNLOAD_URL=$(echo "$RESPONSE" | jq -r '.signed_url // empty')

          # Validate output before running curl
          if [ -z "$DOWNLOAD_URL" ]; then
            echo "ERROR: Failed to retrieve download URL from Cortex API."
            echo "API Response: $RESPONSE"
            exit 1
          fi

          # Download and set permissions
          curl -s -o cortexcli "$DOWNLOAD_URL"
          chmod +x cortexcli
        '''
    }
}

        stage('Run Scan') {
        // Replace the repo-id with your repository like: owner/repo
            steps {
                script {
                    unstash 'source'

                    sh """
                    ./cortexcli \
                      --api-base-url "${env.CORTEX_API_URL}" \
                      --api-key "${env.CORTEX_API_KEY}" \
                      --api-key-id "${env.CORTEX_API_KEY_ID}" \
                      code scan \
                      --directory "\$(pwd)" \
                      --repo-id findromojo/cortex-cloud-lab-jenkins \
                      --branch "main" \
                      --source "JENKINS" \
                      --create-repo-if-missing
                    """
                }
            }
        }
    }
}
