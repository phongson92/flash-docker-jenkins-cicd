pipeline {

  agent none //để agent none để chạy từng agent. Từng stages riêng biệt

  environment {
    DOCKER_IMAGE = "phongson92/flask-docker" // biến môi trường
  }

  stages {
    stage("Test") {
      agent {
          docker {
            image 'python:3.8-slim-buster' //image ko có sẵn user jenkin
            args '-u 0:0 -v /tmp:/root/.cache' //-u 0:0 để khai báo user root ;-v... để mount thư mục cache ra /tmp, khi poetry install thì sẽ lưu vào thư mục cache khi build lại sẽ nhanh hơn
          }
      }
      steps {
        sh "pip install poetry" //install nmp vào hệ thống trước
        sh "poetry install"
        sh "poetry run pytest"
      }
    }

    stage("build") {
      agent { node {label 'master'}}
      environment {
        DOCKER_TAG="${GIT_BRANCH.tokenize('/').pop()}-${GIT_COMMIT.substring(0,7)}"
      } //tạo version của image tên nhánh + build-number + git_commit
      steps {
        sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} . "
        sh "docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest"
        sh "docker image ls | grep ${DOCKER_IMAGE}"
        withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) { //khai báo thông tin login docker hub
            sh 'echo $DOCKER_PASSWORD | docker login --username $DOCKER_USERNAME --password-stdin'
            sh "docker push ${DOCKER_IMAGE}:${DOCKER_TAG}"
            sh "docker push ${DOCKER_IMAGE}:latest"
        }

        //clean to save disk
        sh "docker image rm ${DOCKER_IMAGE}:${DOCKER_TAG}" //xóa image sau khi đã push xong để trống bộ nhớ
        sh "docker image rm ${DOCKER_IMAGE}:latest"
      }
    }
  }

  post {
    success {
      echo "SUCCESSFUL"
    }
    failure {
      echo "FAILED"
    }
  }
}
