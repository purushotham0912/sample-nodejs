pipeline { 
    agent { label 'agent-online' }   // The Jenkins agent label 
 
    environment { 
        APP_DIR = "/home/ubuntu/sample-nodejs" 
    } 
 
    stages { 
        stage('Checkout') { 
            steps { 
                dir("${APP_DIR}") { 
                    git branch: 'main', url: 'https://github.com/etichiranjeevi/sample-nodejs.git' 
                } 
            } 
        } 
 
        stage('Install Dependencies') { 
            steps { 
                dir("${APP_DIR}") { 
                    sh 'npm install && npm install http-errors' 
                } 
            } 
        } 
 
        stage('Run Application') { 
            steps { 
                dir("${APP_DIR}") { 
                    sh ''' 
                        pm2 delete sample-nodejs || true
                        pm2 start ./bin/www --name sample-nodejs
                        pm2 save
                    ''' 
                } 
            } 
        } 
 
        stage('Verify') { 
            steps { 
                dir("${APP_DIR}") { 
                    sh 'ps -ef | grep "node ./bin/www" || true' 
                } 
            } 
        } 
    } 
} 
