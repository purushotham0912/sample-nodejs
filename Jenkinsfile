pipeline { 
    agent { label 'ubuntu-agent' }

    environment { 
        APP_DIR = "/home/ubuntu/sample-nodejs"
        APP_NAME = "sample-node-app"
    }

    stages { 
        stage('Checkout') { 
            steps { 
                dir("${APP_DIR}") { 
                    git branch: 'main', url: 'https://github.com/digitalocean/sample-nodejs.git'
                } 
            } 
        } 

        stage('Install Node.js & npm') {
            steps {
                sh '''
                    sudo apt update -y
                    sudo apt install -y nodejs npm
                    node -v
                    npm -v
                '''
            }
        }

        stage('Install PM2') {
            steps {
                sh '''
                    sudo npm install -g pm2
                    pm2 -v
                '''
            }
        }

        stage('Prepare Node App') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        # Update Node app to listen on all interfaces
                        grep -q "server.listen(port);" ./bin/www && \
                        sed -i "s/server.listen(port);/server.listen(port, '0.0.0.0');/" ./bin/www || true
                    '''
                }
            }
        }

        stage('Create systemd Service') {
            steps {
                sh '''
                    sudo tee /etc/systemd/system/${APP_NAME}.service > /dev/null <<EOL
[Unit]
Description=Node.js Application - ${APP_NAME}
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=${APP_DIR}
ExecStart=/usr/bin/env node ${APP_DIR}/bin/www
Restart=always
Environment=PATH=/usr/bin:/usr/local/bin
Environment=NODE_ENV=production

[Install]
WantedBy=multi-user.target
EOL
                '''
            }
        }

        stage('Start & Enable Service') {
            steps {
                sh '''
                    # Reload systemd to pick up new service
                    sudo systemctl daemon-reload

                    # Enable service to start on boot
                    sudo systemctl enable ${APP_NAME}.service

                    # Start/restart service
                    sudo systemctl restart ${APP_NAME}.service

                    # Check status
                    sudo systemctl status ${APP_NAME}.service --no-pager
                '''
            }
        }

        stage('Verify') { 
            steps { 
                dir("${APP_DIR}") { 
                    sh '''
                        echo "🌀 Checking if port 3000 is listening..."
                        sudo lsof -i :3000 || true

                        echo "🌐 Checking HTTP response via localhost..."
                        curl -I http://localhost:3000 || true
                    '''
                } 
            } 
        } 
    } 
}
pipeline {
    agent { label 'agent-online' }   // Jenkins agent label

    environment {
        APP_DIR = "/home/ubuntu/sample-nodejs"
        APP_NAME = "sample-nodejs"   // PM2 process name (you can change if you want)
    }

    stages {
        stage('Checkout') {
            steps {
                dir("${APP_DIR}") {
                    git branch: 'main', url: 'https://github.com/purushotham0912/sample-nodejs.git'
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                dir("${APP_DIR}") {
                    sh 'npm install'
                }
            }
        }

        stage('Run Application with PM2') {
            steps {
                dir("${APP_DIR}") {
                    sh '''
                        # Stop old instance if running
                        pm2 delete $APP_NAME || true

                        # Start new instance with PM2
                        pm2 start ./bin/www --name $APP_NAME

                        # Save PM2 state (so it survives reboot if needed)
                        pm2 save
                    '''
                }
            }
        }

        stage('Verify') {
            steps {
                sh 'pm2 status $APP_NAME'
            }
        }
    }
}

