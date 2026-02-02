pipeline {
    agent any

    environment {
        // Cambiamos 'latest' por una versión específica (v1.18.0)
        TERRASCAN_IMAGE = 'tenable/terrascan:1.18.0'
        // Montamos directamente la carpeta de AWS para evitar el error de "directorio vacío"
        DOCKER_ARGS = "-v ${WORKSPACE}/terraform/aws:/iac -w /iac"
    }

    stages {
        stage('Limpieza') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: '*/master']],
                    userRemoteConfigs: [[url: 'https://github.com/aabenitez/terragoat.git']] 
                ])
            }
        }

        stage('Scan IaC - Terrascan') {
            steps {
                script {
                    echo "--- Iniciando Escaneo con Terrascan v1.18.0 ---"
                    // Eliminamos el -d porque ya estamos montados en la carpeta correcta
                    sh """
                        docker pull ${TERRASCAN_IMAGE}
                        docker run --rm ${DOCKER_ARGS} ${TERRASCAN_IMAGE} scan \
                        -i terraform \
                        -t aws \
                        --verbose > terrascan_report.txt || true
                        
                        cat terrascan_report.txt
                    """
                }
            }
        }
    }
    
    post {
        always {
            archiveArtifacts artifacts: 'terrascan_report.txt'
            // Limpieza de imagen específica
            sh "docker rmi ${TERRASCAN_IMAGE} || true"
        }
    }
}
