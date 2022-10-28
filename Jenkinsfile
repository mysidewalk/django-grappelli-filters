pipeline {
    agent {
        docker {
            image 'gcr.io/mindmixer-sidewalk/python:3.9'
            args '-u root:root'
        }
    }
    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }
    stages {
        stage('Build') {
            steps {
                sh 'pip install --upgrade pip==22.1.2'
                sh 'pip install . --no-index'
                sh 'chmod -R 777 .'
            }
        }
        stage('Lint') {
            steps {
                // https://github.com/actions/starter-workflows/blob/master/ci/python-package.yml
                sh 'pip install flake8'
                // stop the build if there are Python syntax errors or undefined names
                sh 'flake8 . --count --select=E9,F63,F7,F82 --show-source --statistics'
                // exit-zero treats all errors as warnings. The GitHub editor is 127 chars wide
                sh 'flake8 . --count --exit-zero --max-complexity=10 --max-line-length=127 --statistics'
            }
        }
        stage('Deploy') {
            when { buildingTag() }
            steps {
                sh 'pip install twine wheel'
                sh 'python setup.py bdist_wheel'
                sh 'twine upload dist/*.whl -r mysipypi'
                // make sure the cleanup step is able to delete newly created files
                sh 'chmod -R 777 .'
            }
        }
    }
    post {
        cleanup {
            deleteDir()
        }
        failure {
            script {
                def message = "${currentBuild.fullDisplayName} Failed. (<${env.RUN_DISPLAY_URL}|Open>)"
                slackSend channel: '#jenkins', color: 'danger', message: message
            }
        }
        success {
            script {
                if (currentBuild.previousBuild?.result != 'SUCCESS') {
                  def message = "${currentBuild.fullDisplayName} Back to normal. (<${env.RUN_DISPLAY_URL}|Open>)"
                  slackSend channel: '#jenkins', color: 'good', message: message
                }
            }
        }
    }
}
