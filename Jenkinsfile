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
                    // 1. Pull explícito de la imagen
                    sh "docker pull ${TERRASCAN_IMAGE}"

		    // Verificación en una sola línea, para ver si terrascan puede ver los archivos.
		    sh "docker run --rm --user root -v ${WORKSPACE}:/iac alpine ls -R /iac"

                    // 2. Ejecución con manejo de exit code
                    // Agregamos '|| true' o capturamos el estatus para que el Exit Code 4 no mate el pipeline antes de leer el archivo
                    sh "docker run --rm --user root -v ${WORKSPACE}:/iac ${TERRASCAN_IMAGE} scan -t aws -d /iac -o json > terrascan_result.json || echo 'Escaneo finalizado con hallazgos'"		    

                    // Validar si el archivo existe antes de leerlo
                    if (fileExists("terrascan_result.json")) {
			def reportContent = readFile "terrascan_result.json"

			// Validamos si hay resultados (ajusta según la estructura del JSON de Terrascan)
                        if (reportContent.contains('"high_severity": 0')) {
                            echo "✅ No se encontraron vulnerabilidades críticas."
                        } else {
                            echo "⚠️  Se detectaron vulnerabilidades de severidad alta."
                            currentBuild.result = 'UNSTABLE'
                        }
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
