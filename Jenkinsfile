pipeline {
    agent any

    environment {
        // Version pinning para estabilidad
        TERRASCAN_IMAGE = 'tenable/terrascan:1.18.0'
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
			// 🔍 DEBUG: ver estructura real de archivos
	                sh "echo '--- CONTENIDO DEL WORKSPACE ACTUAL ---'"
	                sh "ls -R ."

                        sh """
			    docker pull ${TERRASCAN_IMAGE}
			    docker run --rm -v \$(pwd):/iac -w /iac \
			    ${TERRASCAN_IMAGE} scan -i . -t aws --recursive > terrascan_report.txt || true
			"""

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
