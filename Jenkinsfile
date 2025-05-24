pipeline {
  agent any

  environment {
    GH_TOKEN = credentials('github-token')
  }

  stages {
    stage('Clean old images') {
      steps {
        script {
          def images = ['fleet_master_main_project', 'fleet_master_auth_project']

          images.each { image ->
            echo "Cleaning old images for ${image}..."
            def imageIds = sh(
              script: "docker images ${image} --format '{{.ID}}' | tail -n +3",
              returnStdout: true
            ).trim().split("\n")

            if (imageIds.size() > 0) {
              imageIds.each { id ->
                sh "docker rmi -f ${id} || true"
              }
            }
          }
        }
      }
    }

    stage('Deploy') {
      steps {
        script {
          def running = sh(script: "docker compose ps -q", returnStdout: true).trim()
          if (running == "") {
            echo "No hay contenedores corriendo. Iniciando todo..."
            sh 'docker compose up -d'
          } else {
            echo "Contenedores en ejecución. Verificando qué servicio actualizar..."
            def commitMsg = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
            echo "Último commit: ${commitMsg}"
            def serviceName = sh(script: "echo \"${commitMsg}\" | sed -n 's/.*update \\([^ ]*\\).*/\\1/p'",
              returnStdout: true
              ).trim()
            
            if (serviceName) {
              echo "Actualizando solo el servicio: ${serviceName}"
              sh "docker compose up -d ${serviceName}"
            } else {
              echo "No se detectó un servicio específico. Haciendo compose up de todo."
              sh "docker compose up -d"
            }
          }
        }
      }
    }
  }
}
