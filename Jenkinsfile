pipeline {
    agent { dockerfile true }

    environment {
        DOCKER_IMAGE = 'ghcr.io/ymir0804/dev-app'
        GHCR_CREDS   = 'ymir0804'
    }
    
    stages {
        stage('Build and Push') {
            steps {
                script {
                    def version = readFile('VERSION').trim()
                    env.DOCKER_TAG = version
                    
                    docker.withRegistry('https://ghcr.io', GHCR_CREDS) {
                        def customImage = docker.build(":", ".")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
    }
}
