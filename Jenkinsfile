pipeline{
    agent any // 어떤 에이전트(실행서버)에서든 실행 가능

    tools{
        maven 'maven 3.9.11' // Jenkins에 등록된 버전 사용

    }

    environment {
        // 배포에 필요한 변수 설정
        DOCKER_IMAGE = "demo-app"
        CONTAINER_NAME = "springboot-container"
        JAR_FILE_NAME = "app.jar"
        PORT = "8081"

        REMOTE_USER = "ec2-user"
        REMOTE_HOST = "15.164.216.201"
        REMOTE_DIR = "/home/ec2-user/deploy"

        SSH_CREDENTIALS_ID = "db284ef7-fff6-48de-98f9-025879ec2fe3"
    }

    stages{
        stage('Git Checkout') {
            steps{
                checkout scm
            }
        }

        stage('Maven Build'){
            steps {
                sh 'mvn clean package -DskipTests' // jenkins에서 리눅스 명령어 실행 방법
            }
        }

        stage('Prepare Jar') {
            steps {
                // 빌드 결과물인 JAR 파일을 지정한 이름 app.jar 으로 복사
                sh 'cp target/demo-0.0.1-SNAPSHOT.jar ${JAR_FILE_NAME}'
            }
        }
        stage('Copy to Remote Server') {
            steps {
                // Jenkins가 원격 서버에 SSH 접속할수 있도록 sshagent 사용
                sshagent(credentials:[env.SSH_CREDENTIALS_ID]){
                    // 원격 서버에 배포 디렉토리 생성 (없으면 새로 만듦)
                    sh "ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} \"mkdir -p ${REMOTE_DIR}\""
                    // JAR 파일과 Dockerfile을 원격 서버에 복사
                    sh "scp -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${JAR_FILE_NAME} Dockerfile ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/"
                }
                
            }            
        }
        stage('Remote Docker Build & Deploy') {
            steps {
                sshagent (credentials: [env.SSH_CREDENTIALS_ID]) {
                    sh """
                    ssh -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null ${REMOTE_USER}@${REMOTE_HOST} << ENDSSH \
                    cd ${REMOTE_DIR} || exit 1 \
                    docker rm -f ${CONTAINER_NAME} || true \
                    docker build -t ${DOCKER_IMAGE} . \
                    docker run -d --name ${CONTAINER_NAME} -p ${PORT}:${PORT} ${DOCKER_IMAGE} \
                    ENDSSH \
                    """
                }
            }
        }
    }
}