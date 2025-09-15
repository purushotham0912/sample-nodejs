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
                    sh 'npm install' 
                } 
            } 
        } 
 
        stage('Run Application') { 
            steps { 
                dir("${APP_DIR}") { 
                    sh ''' 
                        pkill -f "node ./bin/www" || true 
                        nohup npm start > app.log 2>&1 & 
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
