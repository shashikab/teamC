// This is a Jenkins Pipeline script 
pipeline {
    // Define the agent - here 'any' means it will run on any available Jenkins agent (computer)
    agent any

    // Define environment variables if needed (for example, usernames, passwords, etc.)
    environment {
        // In a real project, you would store these as Jenkins credentials securely.
        APIM_USERNAME = 'admin'
        APIM_PASSWORD = 'admin'
    }

    // Stages are like steps in a recipe.
    stages {
        stage('Checkout Code') {
            steps {
                echo "Checking out code from the repository..."
                // This pulls the latest code from your Git repository.
                git url: 'https://github.com/shashikab/teamA.git', branch: 'main'
            }
        }

        stage('Install apictl') {
            steps {
                echo "Installing the apictl tool..."
                // Run a shell command to download and install apictl.
                sh '''
                    # Download the Linux version of apictl (adjust URL/version if needed)
                    curl -L -o apictl.tar.gz "https://github.com/wso2/product-apim-tooling/releases/download/v4.4.1/apictl-4.4.1-darwin-arm64.tar.gz"
                    
                    # Create a temporary folder for extraction
                    mkdir -p /tmp/apictl
                    tar -xzvf apictl.tar.gz -C /tmp/apictl
                    
                    # List files to see where the binary is
                    ls -la /tmp/apictl
                    
                    # Check if the binary is extracted directly or inside a folder.
                    if [ -f "/tmp/apictl/apictl" ]; then
                      echo "Found apictl binary directly."
                      mv /tmp/apictl/apictl ~/apictl
                    elif [ -f "/tmp/apictl/apictl/apictl" ]; then
                      echo "Found apictl binary inside a folder."
                      mv /tmp/apictl/apictl/apictl ~/apictl
                    else
                      echo "apictl binary not found!"
                      exit 1
                    fi
                    
                    # Make the binary executable
                    chmod +x ~/apictl
                    
                    # Add the directory containing the binary to the PATH
                    export PATH=~/apictl:$PATH
                    echo "Installed apictl version:"
                    ~/apictl version
                '''
            }
        }

        stage('Configure APIM Environment') {
            steps {
                echo "Configuring the APIM environment..."
                // This command sets up a connection to the API Manager.
                sh '''
                    # Remove any previous "dev" environment (ignore errors if it doesn't exist)
                    ~/apictl remove env dev || true
                    
                    # Set a timeout variable (optional)
                    export APICLIENT_TIMEOUT=120
                    
                    # Add a new environment called "dev" which points to your APIM instance.
                    # Here we assume APIM is running on localhost. Adjust the URL as required.
                    ~/apictl add env dev --apim https://localhost:9443
                '''
            }
        }

        stage('Login to APIM') {
            steps {
                echo "Logging in to APIM..."
                // Login to APIM securely by piping the password.
                sh '''
                    echo "${APIM_PASSWORD}" | ~/apictl login dev --insecure -u ${APIM_USERNAME} --password-stdin -k
                '''
            }
        }

        stage('Deploy API to Dev') {
            steps {
                echo "Deploying the API project..."
                // This command deploys your API project (assumed to be in the folder named "my-api").
                sh '''
                    ~/apictl import api --file ./my-api --environment dev -k --rotate-revision
                '''
            }
        }

        stage('Test API in Dev') {
            steps {
                echo "Testing the deployed API..."
                // A simple test command that calls your API endpoint.
                sh 'curl -k https://localhost:8243/my-api/1.0.0/hello'
            }
        }
    }

    // After all stages, this block runs regardless of the result.
    post {
        always {
            echo "Jenkins Pipeline execution completed."
        }
    }
}
