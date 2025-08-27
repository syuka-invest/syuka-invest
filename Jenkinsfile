pipeline {
    // agent any: 파이프라인을 실행할 Jenkins 노드를 임의로 선택합니다.
    agent any

    stages {
        // --- 1. Git 리포지토리에서 소스 코드 받아오기 ---
        stage('Checkout from GitHub') {
            steps {
                echo '1. GitHub에서 소스 코드를 가져옵니다.'
                git 'https://github.com/syuka-invest/syuka-invest.git'
            }
        }

        // --- 2. Docker 이미지 빌드 및 컨테이너 실행 ---
        stage('Build and Run Docker Container') {
            steps {
                echo '2. Docker 이미지를 빌드하고 컨테이너를 실행합니다.'
                // dir: 실제 작업할 디렉토리인 'backend/Syuka-invest'로 이동합니다.
                dir('backend/Syuka-invest') {
                    script {
                        // 생성할 도커 이미지의 이름과 태그를 변수로 지정합니다.
                        def imageName = "syuka-invest/backend-test:latest"
                        
                        // 현재 디렉토리의 Dockerfile을 사용하여 이미지를 빌드합니다.
                        sh "docker build -t ${imageName} ."

                        // 만약 이전에 실행했던 같은 이름의 컨테이너가 있다면 중지하고 삭제합니다.
                        sh "docker stop syuka-invest-app || true"
                        sh "docker rm syuka-invest-app || true"

                        // 새로 빌드한 이미지로 컨테이너를 실행합니다.
                        // -d: 백그라운드에서 실행
                        // --name: 컨테이너에 'syuka-invest-app'이라는 이름을 붙여줍니다.
                        // -p 8080:8080: 호스트의 8080 포트와 컨테이너의 8080 포트를 연결합니다.
                        sh "docker run -d --name syuka-invest-app -p 8080:8080 ${imageName}"
                    }
                }
            }
        }
    }
}
