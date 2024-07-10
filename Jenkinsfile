pipeline {
    agent any // 모든 가용 에이전트에서 이 파이프라인을 실행합니다.

    environment {
        AWS_DEFAULT_REGION = 'ap-northeast-2' // AWS 기본 리전 설정
        AWS_ACCOUNT_ID = '975050301857' // AWS 계정 ID 설정
        ECR_REPOSITORY = 'punlo' // ECR 저장소 이름 설정
        IMAGE_TAG = "${BUILD_NUMBER}" // 빌드 번호를 이미지 태그로 사용
        DOCKERFILE_PATH = 'Dockerfile' // Dockerfile 경로 설정
        DOCKER_IMAGE_NAME = 'web-intro' // Docker 이미지 이름 설정
    }

    stages {
        stage('Build and Push to ECR') {
            steps {
                script {
                    // Docker 이미지를 --no-cache 옵션으로 빌드합니다.
                    def customImage = docker.build("${DOCKER_IMAGE_NAME}:${IMAGE_TAG}", "--no-cache=true -f ${DOCKERFILE_PATH} .")

                    // AWS ECR에 로그인합니다.
                    withCredentials([[
                        $class: 'AmazonWebServicesCredentialsBinding',
                        credentialsId: 'AWS_ECR_Credentials', // Jenkins에서 설정한 AWS ECR 자격 증명 ID 입력
                        accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                        secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                    ]]) {
                        // AWS ECR에 로그인합니다.
                        def ecrLogin = sh(script: "aws ecr get-login-password --region ${AWS_DEFAULT_REGION}", returnStdout: true).trim()
                        sh "docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com <<< '${ecrLogin}'"
                        
                        // Docker 이미지를 태깅합니다.
                        sh "docker tag ${DOCKER_IMAGE_NAME}:${IMAGE_TAG} ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                        
                        // Docker 이미지를 ECR에 푸시합니다.
                        sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${ECR_REPOSITORY}:${IMAGE_TAG}"
                    }
                }
            }
        }
    }

    post {
        success {
            echo "빌드 및 ECR 푸시 성공, 이미지 버전: ${IMAGE_TAG}" // 빌드 및 푸시가 성공하면 메시지를 출력합니다.
        }
        failure {
            echo '빌드 또는 ECR 푸시 실패' // 빌드 또는 푸시가 실패하면 메시지를 출력합니다.
        }
    }
}
​
