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
                    // 1. Generamos el reporte con sudo si es necesario para escribir en el host
                    // Usamos -f json para que sea fácil de leer después
                    sh 'sudo docker run --rm -v ${WORKSPACE}/terraform:/iac tenable/terrascan:latest scan -t aws -d . -o json > terrascan_result.json'

                    // 2. Cambiamos el dueño del archivo a jenkins para que readFile no tenga problemas de permisos
                    sh 'sudo chown jenkins:jenkins terrascan_result.json'

                    // 3. Usamos readFile para cargar el contenido en una variable de Groovy
                    def reportContent = readFile "terrascan_result.json"

                    // 4. (Opcional) Procesar el contenido
                    if (reportContent.contains('"low_severity": 0')) {
                        echo "¡Excelente! No se encontraron vulnerabilidades de severidad baja."
                    } else {
                        echo "Se detectaron hallazgos en el reporte."
                    }

                    // Mostramos un extracto en los logs de Jenkins
                    echo "Contenido del reporte: ${reportContent}"  
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'terraform/terrascan_report.txt', fingerprint: true, allowEmptyArchive: true
            echo "Eliminando imagen ${TERRASCAN_IMAGE}..."
            sh "docker rmi ${TERRASCAN_IMAGE} || true"
        }
    }
}
