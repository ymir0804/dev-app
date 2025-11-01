pipeline {
    // Dockerfile을 사용해 빌드 환경을 구성하도록 지정
    agent { dockerfile true }

    environment {
        DOCKER_IMAGE = 'ghcr.io/ymir0804/dev-app'
        GHCR_CREDS   = 'ymir0804' // Jenkins에 등록된 Credential ID
    }
    
    stages {
        // 'agent { dockerfile true }'가 체크아웃을 자동으로 수행하므로, 별도 Checkout Stage는 불필요합니다.

        stage('Build and Push') {
            steps {
                script {
                    def version = readFile('VERSION').trim()
                    env.DOCKER_TAG = version
                    
                    docker.withRegistry('https://ghcr.io', GHCR_CREDS) {
                        def customImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", ".")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
    }
    post {
        always {
            // 에이전트에 남은 이미지 정리
            sh "docker rmi ${DOCKER_IMAGE}:${env.DOCKER_TAG} || true"
            sh "docker rmi ${DOCKER_IMAGE}:latest || true"
        }
    }
}

