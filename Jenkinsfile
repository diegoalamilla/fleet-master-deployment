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
                 sh '''
                # Arranca ngrok para cada servicio (en segundo plano)
                ngrok http 8088 > ngrok_fleet_main.log &
                ngrok http 8087 > ngrok_fleet_auth.log &

                # Espera un poco para que ngrok exponga los túneles
                sleep 5

                # Obtiene las URL públicas usando la API de ngrok
                FLEET_MAIN_URL=$(curl -s http://localhost:4040/api/tunnels | jq -r '.tunnels[] | select(.config.addr=="http://localhost:8088") | .public_url')
                FLEET_AUTH_URL=$(curl -s http://localhost:4040/api/tunnels | jq -r '.tunnels[] | select(.config.addr=="http://localhost:8087") | .public_url')

                echo " Fleet Master Main público: $FLEET_MAIN_URL"
                echo " Fleet Master Auth público: $FLEET_AUTH_URL"
                '''
            }
        }
    }
  }
}
