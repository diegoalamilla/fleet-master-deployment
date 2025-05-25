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
              sh "docker compose up -d --force-recreate ${serviceName}"
            } else {
              echo "No se detectó un servicio específico. Haciendo compose up de todo."
              sh "docker compose up -d"
            }
          }
        }
      }
    }
    stage ('Get Public Url'){
        steps{
            script{

                def ngrok = 'C:\\ngrok\\ngrok.exe'
                 
               sh '''
            # Exponer fleet-main
            ngrok http 8088 > ngrok_fleet_main.log &
            sleep 5
            FLEET_MAIN_URL=$(grep -o 'url=https://[^ ]*' ngrok_fleet_main.log | head -n 1 | cut -d= -f2)
            echo "FLEET_MAIN_URL=$FLEET_MAIN_URL"
            echo "FLEET_MAIN_URL=$FLEET_MAIN_URL" >> .env

            # Exponer fleet-auth
            ngrok http 8087 > ngrok_fleet_auth.log &
            sleep 5
            FLEET_AUTH_URL=$(grep -o 'url=https://[^ ]*' ngrok_fleet_auth.log | head -n 1 | cut -d= -f2)
            echo "FLEET_AUTH_URL=$FLEET_AUTH_URL"
            echo "FLEET_AUTH_URL=$FLEET_AUTH_URL" >> .env

            # Confirmar URLs finales
            echo "URLs públicas disponibles:"
            cat .env
          '''
            }
        }
    }
  }
}
