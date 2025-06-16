pipeline {
    agent any
    environment {
        VENV_DIR = 'selenium-env'
        DISPLAY = ':99'
    }
    stage('Install System Dependencies') {
    steps {
        sh '''
            apt-get update -y

            # Install Chrome and dependencies (revised)
            apt-get install -y \
                wget \
                gnupg \
                unzip \
                curl \
                xvfb \
                fonts-liberation \
                libappindicator3-1 \
                libatk-bridge2.0-0 \
                libdrm2 \
                libgtk-3-0 \
                libnspr4 \
                libnss3 \
                libx11-xcb1 \
                libxcomposite1 \
                libxdamage1 \
                libxrandr2 \
                xdg-utils \
                libxss1 \
                libxcb1 \
                libgbm1 \
                libu2f-udev \
                libvulkan1

            # Install Chrome
            if ! command -v google-chrome &> /dev/null; then
                wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add -
                echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
                apt-get update -y
                apt-get install -y google-chrome-stable
            fi

            # Install ChromeDriver
            CHROME_VERSION=$(google-chrome --version | awk '{print $3}' | cut -d'.' -f1-3)
            CHROMEDRIVER_VERSION=$(curl -s "https://chromedriver.storage.googleapis.com/LATEST_RELEASE_${CHROME_VERSION}")
            wget -O /tmp/chromedriver.zip "https://chromedriver.storage.googleapis.com/${CHROMEDRIVER_VERSION}/chromedriver_linux64.zip"
            unzip -o /tmp/chromedriver.zip -d /usr/local/bin/
            chmod +x /usr/local/bin/chromedriver
            rm /tmp/chromedriver.zip
        '''
    }
}

        stage('Checkout Test Repo') {
            steps {
                checkout scm
            }
        }
        stage('Set Up Python Environment') {
            steps {
                sh '''
                    python3 -m venv $VENV_DIR
                    . $VENV_DIR/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        stage('Start Virtual Display') {
            steps {
                sh '''
                    # Start Xvfb for headless display
                    Xvfb :99 -screen 0 1920x1080x24 > /dev/null 2>&1 &
                    echo $! > /tmp/xvfb.pid
                    sleep 3
                '''
            }
        }
        stage('Run Selenium Tests') {
            steps {
                sh '''
                    export DISPLAY=:99
                    . $VENV_DIR/bin/activate
                    python3 test_app.py
                '''
            }
        }
    }
    post {
        always {
            sh '''
                # Stop Xvfb if running
                if [ -f /tmp/xvfb.pid ]; then
                    kill $(cat /tmp/xvfb.pid) || true
                    rm -f /tmp/xvfb.pid
                fi
                
                # Clean up virtual environment
                rm -rf $VENV_DIR
            '''
        }
        success {
            echo '✅ All tests passed successfully!'
        }
        failure {
            echo '❌ Some tests failed. Check the logs above for details.'
        }
    }
}
