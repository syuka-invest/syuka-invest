pipeline {
    agent any

    // --- 환경 변수 설정 ---
    environment {
        // Jenkins Credentials에 등록된 Docker Hub ID
        DOCKER_HUB_CREDENTIALS_ID = 'syuka-invest-backend-dockerhub' 
        // Docker Hub 사용자명 / 이미지 이름
        DOCKER_IMAGE_NAME = 'blueconecell/syukainvest-backend' 
        // 백엔드 프로젝트 경로
        BACKEND_DIR = 'backend/Syuka-invest'
    }

    stages {
        // --- 1. Gradle 빌드 ---
        stage('Gradle Build') {
            steps {
                echo 'Gradle 빌드를 시작합니다.'
                dir(BACKEND_DIR) {
                    // gradlew 실행 권한 부여
                    sh 'chmod +x gradlew'
                    // Gradle 빌드 실행 (테스트 포함)
                    sh './gradlew build'
                }
            }
        }

        // --- 2. Docker 이미지 빌드 ---
        stage('Build Docker Image') {
            steps {
                echo 'Docker 이미지를 빌드합니다.'
                dir(BACKEND_DIR) {
                    script {
                        // 빌드 번호를 태그로 사용하여 이미지 버전 관리
                        def imageTag = "${env.BUILD_NUMBER}"
                        sh "docker build -t ${DOCKER_IMAGE_NAME}:${imageTag} ."
                        // latest 태그도 함께 추가
                        sh "docker tag ${DOCKER_IMAGE_NAME}:${imageTag} ${DOCKER_IMAGE_NAME}:latest"
                    }
                }
            }
        }

        // --- 3. Docker Hub에 이미지 푸시 ---
        stage('Push to Docker Hub') {
            steps {
                echo "Docker 이미지를 ${DOCKER_IMAGE_NAME}에 푸시합니다."
                // withCredentials 블록으로 안전하게 Docker Hub 로그인 정보 사용
                withCredentials([usernamePassword(credentialsId: DOCKER_HUB_CREDENTIALS_ID, usernameVariable: 'DOCKER_HUB_USERNAME', passwordVariable: 'DOCKER_HUB_PASSWORD')]) {
                    sh "echo ${DOCKER_HUB_PASSWORD} | docker login -u ${DOCKER_HUB_USERNAME} --password-stdin"
                    sh "docker push ${DOCKER_IMAGE_NAME}:${env.BUILD_NUMBER}"
                    sh "docker push ${DOCKER_IMAGE_NAME}:latest"
                }
            }
        }

        // --- 4. Docker Compose로 배포 ---
        stage('Deploy') {
            steps {
                echo 'Docker Compose로 배포를 시작합니다.'
                dir(BACKEND_DIR) {
                    // 최신 이미지를 pull 받음
                    sh "docker-compose -f docker-compose-infra.yml -f docker-compose-backend.yml pull backend"
                    // 인프라와 백엔드 컨테이너 실행 (변경된 서비스만 재시작)
                    sh "docker-compose -f docker-compose-infra.yml -f docker-compose-backend.yml up -d"
                }
            }
        }
    }

    // --- 빌드 후 정리 작업 ---
    post {
        always {
            echo '파이프라인이 종료되었습니다. Docker 로그아웃을 실행합니다.'
            sh 'docker logout'
        }
    }
}