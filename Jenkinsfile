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
                        sh "docker pull ${TERRASCAN_IMAGE}"
                
                        sh "docker run --rm --user root -v \$(pwd):/iac -w /iac ${TERRASCAN_IMAGE} scan -t aws -d . --recursive > terrascan_report.txt || true"
                        
                        echo "--- REPORTE GENERADO ---"

                        // Usamos Docker para leer el archivo como root
                        sh "docker run --rm --user root -v \$(pwd):/iac -w /iac alpine cat terrascan_report.txt"

                        // Cambiamos el dueño para que Jenkins pueda procesar el artefacto en el bloque post
                        sh "docker run --rm --user root -v \$(pwd):/iac -w /iac alpine chown \$(id -u):\$(id -g) terrascan_report.txt || true"
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
