pipeline {
    agent any

    stages {
        // --- Docker 이미지 빌드 및 컨테이너 실행 ---
        stage('Build and Run Docker Container') {
            steps {
                echo 'Docker 이미지를 빌드하고 컨테이너를 실행합니다.'
                dir('backend/Syuka-invest') {
                    script {
                        def imageName = "syuka-invest/backend-test:latest"
                        
                        sh "docker build -t ${imageName} ."

                        sh "docker stop syuka-invest-app || true"
                        sh "docker rm syuka-invest-app || true"

                        sh "docker run -d --name syuka-invest-app -p 8080:8080 ${imageName}"
                    }
                }
            }
        }
    }
}