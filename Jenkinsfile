pipeline {
    agent any

    environment {
        // Definimos la imagen oficial de Terrascan
        TERRASCAN_IMAGE = 'tenable/terrascan:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/aabenitez/terragoat.git', branch: 'master'
            }
        }

        stage('Pull Terrascan Image') {
            steps {
                echo "Descargando imagen de Terrascan..."
                sh "docker pull ${TERRASCAN_IMAGE}"
            }
        }

	stage('Security Scan') {
            steps {
                script {
                    echo "Escaneando subdirectorio terraform/aws/..."
                    // Ejecutamos el contenedor mapeando el directorio actual
                    // El comando '|| true' evita que el pipeline se detenga si encuentra fallos, 
                    // permitiendo que el stage de limpieza siempre se ejecute.
                    sh """
                        docker run --rm \
                            -v ${WORKSPACE}:/iac \
                            -w /iac \
                        ${TERRASCAN_IMAGE} scan -t aws -i terraform -d terraform/aws > terrascan_report.txt || true    
                    """                
                }
            }
        }
    }

    post {
        always {
            // Guardamos el reporte antes de borrar cualquier rastro
            archiveArtifacts artifacts: 'terrascan_report.txt', fingerprint: true
            
            echo "Limpiando el espacio de trabajo y eliminando imagen Docker..."
            // Borramos la imagen para no ocupar espacio en el nodo de Jenkins
            sh "docker rmi ${TERRASCAN_IMAGE} || true"
            
            // Opcional: limpiar archivos temporales
            deleteDir()
        }
        success {
            echo "Análisis completado exitosamente."
        }
        failure {
            echo "El pipeline ha fallado. Revisar los logs."
        }
    }
}
