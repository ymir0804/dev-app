pipeline {
    agent {
        kubernetes {
            label 'kubectl-agent' // 2단계에서 생성한 템플릿 라벨
        }
    }

    // ... (environment 블록은 동일)

    stages {
        stage('Trigger Shipwright Build') {
            // 'kubectl' 컨테이너를 지정하여 해당 컨테이너 내부에서 스텝을 실행
            container('kubectl') {
                steps {
                    script {
                        // ... (buildYaml 생성 로직)
                        def buildYaml = """..."""
                        writeFile file: 'shipwright-build.yaml', text: buildYaml

                        // 이제 이 sh 명령어는 kubectl 컨테이너 안에서 실행됩니다.
                        sh "kubectl apply -f shipwright-build.yaml -n jenkins"
                    }
                }
            }
        }
        stage('Monitor Build Status') {
            container('kubectl') { // 모니터링 단계도 kubectl 컨테이너에서 실행
                steps {
                    // ... (timeout 및 waitUntil 로직)
                }
            }
        }
    }
    post {
        always {
            container('kubectl') { // 정리 단계도 kubectl 컨테이너에서 실행
                echo "Cleaning up Shipwright Build resource..."
                sh "kubectl delete build ${env.BUILD_NAME} -n jenkins --ignore-not-found=true"
            }
        }
    }
}
