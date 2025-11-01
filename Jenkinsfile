pipeline {
    // 새로 추가한 Pod 템플릿을 사용하도록 지정
    agent {
        kubernetes {
            label 'docker-agent'
            defaultContainer 'jnlp' // 기본 작업은 jnlp 컨테이너에서 수행
        }
    }

    environment {
        DOCKER_IMAGE = 'ghcr.io/ymir0804/dev-app'
        GHCR_CREDS   = 'ymir0804'
    }

    stages {
        stage('Checkout SCM') {
            steps {
                // 체크아웃은 jnlp 컨테이너에서 수행
                checkout scm
            }
        }

        stage('Build and Push') {
            steps {
                // Docker 관련 작업은 'docker' 컨테이너에서 실행
                container('docker') {
                    script {
                        def version = readFile('VERSION').trim()
                        env.DOCKER_TAG = version

                        docker.withRegistry('https://ghcr.io', GHCR_CREDS) {
                            // 이제 docker 명령어를 사용할 수 있습니다.
                            def customImage = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}", ".")
                            customImage.push()
                            customImage.push('latest')
                        }
                    }
                }
            }
        }
    }
}

