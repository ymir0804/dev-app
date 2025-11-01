pipeline {
    // Kubernetes 플러그인에 'kubectl-agent' 라벨로 정의된 Pod 템플릿을 사용
    agent any

    environment {
        BUILD_NAME   = "dev-app-build-${env.BUILD_NUMBER}"
        OUTPUT_IMAGE = "ghcr.io/ymir0804/dev-app:${env.BUILD_NUMBER}"
        GIT_REPO     = "https://github.com/ymir0804/dev-app.git"
        // GHCR 인증 정보 Secret 이름
        GHCR_SECRET  = "ghcr-secret"
        // Shipwright 빌드 전략 이름
        BUILD_STRATEGY = "buildah" 
    }

    stages {
        stage('Trigger Shipwright Build') {
            // 1. 'steps' 블록을 추가합니다.
            steps {
                // 2. 'container' 스텝을 'steps' 블록 안으로 이동시킵니다.
                container('kubectl') {
                    script {
                        // Shipwright Build 리소스 YAML을 동적으로 생성
                        def buildYaml = """
apiVersion: shipwright.io/v1beta1
kind: Build
metadata:
  name: ${BUILD_NAME}
spec:
  source:
    git:
      url: ${GIT_REPO}
  strategy:
    name: ${BUILD_STRATEGY}
    kind: ClusterBuildStrategy
  output:
    image: ${OUTPUT_IMAGE}
    pushSecret: ${GHCR_SECRET}
"""
                        // 생성된 YAML을 클러스터에 적용하여 빌드 시작
                        writeFile file: 'shipwright-build.yaml', text: buildYaml
                        sh "kubectl apply -f shipwright-build.yaml -n jenkins"
                    }
                }
            }
        }

        stage('Monitor Build Status') {
            steps {
                container('kubectl') {
                    // 빌드가 완료될 때까지 최대 15분 대기
                    timeout(time: 15, unit: 'MINUTES') {
                        waitUntil {
                            script {
                                def condition = sh(script: "kubectl get build ${BUILD_NAME} -n jenkins -o jsonpath='{.status.conditions[?(@.type==\"Succeeded\")].status}'", returnStdout: true).trim()
                                if (condition == 'True') {
                                    echo "Shipwright build with ${BUILD_STRATEGY} succeeded!"
                                    return true
                                } else if (condition == 'False') {
                                    def taskRunName = sh(script: "kubectl get build ${BUILD_NAME} -n jenkins -o jsonpath='{.status.latestTaskRunRef}'", returnStdout: true).trim()
                                    error "Shipwright build failed. Check logs of Tekton TaskRun: ${taskRunName}"
                                }
                                echo "Build in progress... waiting."
                                sleep(10)
                                return false
                            }
                        }
                    }
                }
            }
        }
    }
    
    post {
        always {
            // post 블록 안의 container 스텝도 steps 블록으로 감싸야 합니다.
            steps {
                container('kubectl') {
                    echo "Cleaning up Shipwright Build resource..."
                    sh "kubectl delete build ${BUILD_NAME} -n jenkins --ignore-not-found=true"
                }
            }
        }
    }
}
