pipeline {     
    agent any
        
    environment {
        PYENV_HOME = "${WORKSPACE}/welcome/app/flask-volt-dashboard/.pyenv"
        PROJECT_HOME = "${WORKSPACE}/welcome/app/flask-volt-dashboard"
        FLASK_APP = 'run.py'
        FLASK_ENV = 'development'
    }

    stages {
        stage('Clone Flask Project') {
            steps {
                git branch: 'jenkins-workshop', url: 'https://github.com/yanivomc/devopshift-welcome.git'
            }
        }

        stage('Setup Python Environment and Install Dependencies') {
            steps {
                dir('welcome/app/flask-volt-dashboard') {
                    script {
                        sh '''
                            rm -rf .pyenv
                            virtualenv .pyenv
                            . .pyenv/bin/activate
                            pip install --upgrade pip
                            pip install setuptools
                            pip install -r requirements.txt
                        '''
                    }
                }
            }
        }

        stage('Run Flask Application') {
            steps {
                dir("${PROJECT_HOME}") {
                    script {
                        sh '''#!/bin/bash
                        source $PYENV_HOME/bin/activate
                        tasks=$(pgrep -f "flask run")
                        if [ -n "$tasks" ]; then
                            echo "Stopping existing Flask application..."
                            for pid in $tasks; do
                                kill -9 $pid
                                echo "Killed process $pid"
                            done
                        fi

                        echo "Starting Flask application on 0.0.0.0:5001..."
                        nohup flask run --host=0.0.0.0 --port=5001 > flask_app.log 2>&1 &
                        '''
                    }
                }
            }
        }

        stage('Verify Application') {
            steps {
                dir("${PROJECT_HOME}") {
                    script {
                        sleep 5
                        def tasks = sh(script: "pgrep -f 'flask run'", returnStatus: true) ? '' : sh(script: "pgrep -f 'flask run'", returnStdout: true).trim()
                        if (!tasks) {
                            echo "There is a problem with our flask application - printing log below"
                            sh 'cat flask_app.log'
                            error "Flask application is not running!"
                        } else {
                            echo "Flask application is running successfully."
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            dir("${PROJECT_HOME}") {
                echo 'Cleaning up workspace...'
                sh 'rm -rf $PYENV_HOME'
            }
        }
    }
}
