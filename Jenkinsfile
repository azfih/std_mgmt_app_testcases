pipeline {
  agent any

  environment {
    VENV_DIR = 'selenium-env'
  }

  stages {
    stage('Checkout Test Repo') {
      steps {
        checkout scm
      }
    }

    stage('Set Up Python Env') {
      steps {
        sh '''
          python3 -m venv $VENV_DIR
          . $VENV_DIR/bin/activate
          pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Run Selenium Tests') {
      steps {
        sh '''
          . $VENV_DIR/bin/activate
          python3 test_app.py
        '''
      }
    }
  }

  post {
    always {
      echo 'Cleaning up virtual environment...'
      sh 'rm -rf $VENV_DIR'
    }
    success {
      echo '✅ Tests Passed!'
    }
    failure {
      echo '❌ Some tests failed.'
    }
  }
}
