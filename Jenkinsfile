pipeline {
    agent any

    environment {
        DOCKER_ID = "dstdockerhub"
        DOCKER_IMAGE = "datascientestapi"
        DOCKER_TAG = "v.${BUILD_ID}.0"
    }

    stages {

        stage('Building') {
            steps {
                echo 'Installing dependencies...'
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Testing') {
            steps {
                echo 'Running tests...'
                sh 'venv/bin/python -m unittest'
            }
        }

        stage('Deploying') {
            steps {
                script {
                    echo 'Building Docker image and deploying container...'

                    sh '''
                    docker rm -f jenkins || true

                    docker build \
                        -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .

                    docker run -d \
                        -p 8000:8000 \
                        --name jenkins \
                        $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }

        stage('User Acceptance') {
            steps {
                input(
                    message: 'Proceed to push image to DockerHub and merge?',
                    ok: 'Yes'
                )
            }
        }
stage('Pushing and Merging') {
            parallel {

                stage('Pushing Image') {
                    environment {
                        DOCKERHUB_CREDENTIALS = credentials('docker_jenkins')
                    }

                    steps {
                        echo 'Logging into DockerHub...'

                        sh '''
                        echo $DOCKERHUB_CREDENTIALS_PSW | \
                        docker login \
                        -u $DOCKERHUB_CREDENTIALS_USR \
                        --password-stdin
                        '''

                        echo 'Pushing Docker image...'

                        sh '''
                        docker push \
                        $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                        '''
                    }
                }

                stage('Merging') {
                    steps {
                        echo 'Merging done'
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Cleaning up...'
            sh 'docker logout || true'
        }

        success {
            echo 'Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}
