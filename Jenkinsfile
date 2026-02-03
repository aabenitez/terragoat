pipeline {
    agent any

    environment {
        // Version pinning para estabilidad
        TERRASCAN_IMAGE = 'tenable/terrascan:latest'
    }

    stages {
        stage('Limpieza Inicial') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                // Jenkins descargará el repo en su carpeta de WORKSPACE automáticamente
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
                    // Se utiliza --user root para garantizar permisos de escritura en el volumen
                    // Se elimina 'sudo' ya que el entorno de Jenkins no lo tiene instalado
                    sh "docker run --rm --user root -v ${WORKSPACE}/terraform:/iac ${TERRASCAN_IMAGE} scan -t aws -d /iac -o json > terrascan_result.json"

                    // Validar si el archivo existe antes de leerlo
                    if (fileExists("terrascan_result.json")) {
                        def reportContent = readFile "terrascan_result.json"
                        
                        if (reportContent.contains('"low_severity": 0')) {
                            echo "¡Excelente! No se encontraron vulnerabilidades de severidad baja."
                        } else {
                            echo "Se detectaron hallazgos en el reporte."
                        }
                        
                        echo "Contenido del reporte cargado correctamente."
                    } else {
                        error "El archivo terrascan_result.json no fue generado."
                    }
                }
            }
        }
    }

    post {
        always {
            // Se actualizó el nombre del artefacto para coincidir con el archivo generado
            archiveArtifacts artifacts: 'terrascan_result.json', fingerprint: true, allowEmptyArchive: true
            echo "Eliminando imagen ${TERRASCAN_IMAGE}..."
            sh "docker rmi ${TERRASCAN_IMAGE} || true"
        }
    }
}
