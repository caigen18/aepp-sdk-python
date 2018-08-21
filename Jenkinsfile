pipeline {
  agent {
    dockerfile {
      filename 'Dockerfile.ci'
      args '-v /etc/group:/etc/group:ro ' +
           '-v /etc/passwd:/etc/passwd:ro ' +
           '-v /var/lib/jenkins:/var/lib/jenkins ' +
           '-v /usr/bin/docker:/usr/bin/docker:ro ' +
           '--network=host'
    }
  }

  environment {
    DOCKER_COMPOSE = "docker-compose -p ${env.BUILD_TAG} -H 127.0.0.1:2376"
  }

  stages {

    stage('Test') {
      steps {
          withCredentials([usernamePassword(credentialsId: 'genesis-wallet',
                                          usernameVariable: 'WALLET_PUB',
                                          passwordVariable: 'WALLET_PRIV')]) {
          sh "${env.DOCKER_COMPOSE} run sdk flake8"
          sh "${env.DOCKER_COMPOSE} run sdk pytest --junitxml test-results.xml aeternity/tests"
        }
      }
    }

    stage('Publish-Test') {
      when {
        beforeAgent true
        branch 'feature/epoch-0.20.0'
      }
      steps {
        withCredentials([
          usernamePassword(credentialsId: 'test_pypi',
                                          usernameVariable: 'TWINE_USERNAME',
                                          passwordVariable: 'TWINE_PASSWORD'),
          string(credentialsId: 'pypi_test_repo_url', variable: 'TWINE_REPOSITORY_URL')]) {
            sh '''make clean && make build && make publish'''
        }
      }
    }

    stage('Publish') {
      when {
        beforeAgent true
        branch 'maaster'
      }
      steps {
        withCredentials([
          usernamePassword(credentialsId: 'pypi',
                                          usernameVariable: 'TWINE_USERNAME',
                                          passwordVariable: 'TWINE_PASSWORD'),
          string(credentialsId: 'pypi_repo_url', variable: 'TWINE_REPOSITORY_URL')]) {
            sh '''make clean && make build && make publish'''
        }
      }
    }
  }

  post {
    always {
      junit 'test-results.xml'
      sh "${env.DOCKER_COMPOSE} run sdk git clean -fdx"
      sh "${env.DOCKER_COMPOSE} down -v ||:"
    }
  }
}