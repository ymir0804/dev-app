pipeline {
    // 이 파이프라인의 실행 환경을 Dockerfile을 기반으로 구축
    agent { dockerfile true }

    environment {
        DOCKER_IMAGE = 'ghcr.io/ymir0804/dev-app'
        GHCR_CREDS   = 'ymir0804'
    }
    
    stages {
        // 'agent { dockerfile true }'가 체크아웃을 자동으로 수행하므로, 별도 Checkout Stage는 불필요합니다.

        stage('Build and Push') {
            steps {
                script {
                    def version = readFile('VERSION').trim()
                    env.DOCKER_TAG = version
                    
                    // 이 스텝은 'docker-workflow' 플러그인이 제공하는 'docker' 객체를 사용합니다.
                    docker.withRegistry('https://ghcr.io', GHCR_CREDS) {
                        def customImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", ".")
                        customImage.push()
                        customImage.push('latest')
                    }
                }
            }
        }
    }
}

