pipeline {
    agent any
    environment {
        VENV_DIR = 'selenium-env'
        DISPLAY = ':99'
    }
    stages {
        stage('Install System Dependencies') {
            steps {
                sh '''
                    sudo apt-get update -y

                    # Install Chrome and dependencies
                    sudo apt-get install -y \
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
                        libvulkan1 \
                        jq

                    # Install Chrome (if not already installed)
                    if ! command -v google-chrome &> /dev/null; then
                        wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | sudo apt-key add -
                        echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" | sudo tee /etc/apt/sources.list.d/google-chrome.list
                        sudo apt-get update -y
                        sudo apt-get install -y google-chrome-stable
                    fi

                    # Get Chrome version
                    CHROME_VERSION=$(google-chrome --version | awk '{print $3}' | cut -d'.' -f1-3)
                    echo "Chrome version: $CHROME_VERSION"

                    # Method 1: Use Chrome for Testing API (recommended for newer versions)
                    echo "Attempting to download ChromeDriver using Chrome for Testing API..."
                    CHROMEDRIVER_URL=$(curl -s "https://googlechromelabs.github.io/chrome-for-testing/known-good-versions-with-downloads.json" | \
                        jq -r --arg version "$CHROME_VERSION" '.versions[] | select(.version | startswith($version)) | .downloads.chromedriver[] | select(.platform=="linux64") | .url' | \
                        head -1)
                    
                    if [ -n "$CHROMEDRIVER_URL" ] && [ "$CHROMEDRIVER_URL" != "null" ]; then
                        echo "Found ChromeDriver URL: $CHROMEDRIVER_URL"
                        wget -O /tmp/chromedriver.zip "$CHROMEDRIVER_URL"
                        sudo unzip -o /tmp/chromedriver.zip -d /tmp/
                        sudo mv /tmp/chromedriver-linux64/chromedriver /usr/local/bin/chromedriver
                        sudo chmod +x /usr/local/bin/chromedriver
                        sudo rm -rf /tmp/chromedriver.zip /tmp/chromedriver-linux64
                        echo "ChromeDriver installed successfully via Chrome for Testing API"
                    else
                        echo "Chrome for Testing API failed, trying alternative method..."
                        
                        # Method 2: Use the latest stable release
                        LATEST_CHROMEDRIVER_URL=$(curl -s "https://googlechromelabs.github.io/chrome-for-testing/last-known-good-versions.json" | \
                            jq -r '.channels.Stable.downloads.chromedriver[] | select(.platform=="linux64") | .url')
                        
                        if [ -n "$LATEST_CHROMEDRIVER_URL" ] && [ "$LATEST_CHROMEDRIVER_URL" != "null" ]; then
                            echo "Using latest stable ChromeDriver: $LATEST_CHROMEDRIVER_URL"
                            wget -O /tmp/chromedriver.zip "$LATEST_CHROMEDRIVER_URL"
                            sudo unzip -o /tmp/chromedriver.zip -d /tmp/
                            sudo mv /tmp/chromedriver-linux64/chromedriver /usr/local/bin/chromedriver
                            sudo chmod +x /usr/local/bin/chromedriver
                            sudo rm -rf /tmp/chromedriver.zip /tmp/chromedriver-linux64
                            echo "ChromeDriver installed successfully via latest stable"
                        else
                            echo "All ChromeDriver download methods failed!"
                            exit 1
                        fi
                    fi

                    # Verify installation
                    chromedriver --version
                    google-chrome --version
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
            echo '✅ All tests passed successfully!!'
        }
        failure {
            echo '❌ Some tests failed. Check the logs above for details.'
        }
    }
}
