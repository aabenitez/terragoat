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
		    // Entramos a la carpeta de AWS y ejecutamos Docker desde ahí
                    dir('terraform') {
                        sh "docker pull tenable/terrascan:latest"
                
                        // Usamos comillas simples para el comando sh para evitar problemas de escape
                        // y aseguramos que el path sea el actual '.'
                        sh "docker run --rm --user root -v \$(pwd):/iac -w /iac tenable/terrascan:latest scan -t aws -d . --recursive > terrascan_report.txt 2>&1 || true"

                        echo "--- REPORTE GENERADO ---"
                        sh "cat terrascan_report.txt"
                    }    
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
