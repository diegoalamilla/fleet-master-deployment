pipeline {
  agent any

  environment {
    GH_TOKEN = credentials('github-token')
  }

  stages {
    stage('Clone Deployment Repo') {
      steps {
        sh '''
          rm -rf fleet-master-deployment
          git clone https://x-access-token:${GH_TOKEN}@github.com/diegoalamilla/fleet-master-deployment.git
          cd fleet-master-deployment
          git pull
        '''
      }
    }

    stage('Clean old images') {
      steps {
        script {
          // Configura los nombres de tus imágenes
          def images = ['fleet_master_main_project', 'fleet_master_auth_project']

          images.each { image ->
            echo "Cleaning old images for ${image}..."
            // Obtiene los IDs de las imágenes ordenadas por antigüedad
            def imageIds = sh(
              script: "docker images ${image} --format '{{.ID}}' | tail -n +3",
              returnStdout: true
            ).trim().split("\n")

            // Borra las imágenes viejas (a partir de la tercera más antigua)
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
        dir('fleet-master-deployment') {
          script {
            def running = sh(script: "docker compose ps -q", returnStdout: true).trim()
            if (running == "") {
              echo "No hay contenedores corriendo. Iniciando todo..."
              sh 'docker compose up -d'
            } else {
              echo "Contenedores en ejecución. Verificando qué servicio actualizar..."
              def commitMsg = sh(script: "git log -1 --pretty=%B", returnStdout: true).trim()
              echo "Último commit: ${commitMsg}"
              def serviceName = sh(script: "echo \"${commitMsg}\" | grep -oP '(?<=update )[^ ]+'", returnStdout: true).trim()
              
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
}
