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

		// Debug: Listar archivos para ver dónde estamos realmente
                sh "ls -R ${WORKSPACE}/terraform/aws"	
            }
        }

	stage('Security Scan') {
	    steps {
	        script {
	            sh "docker pull ${TERRASCAN_IMAGE}"
            
	            // Usamos -u para evitar problemas de permisos y montamos el workspace
	            sh """
	                docker run --rm \
	                    -u \$(id -u):\$(id -g) \
	                    -v ${WORKSPACE}:/project \
	                    -w /project \
	                    ${TERRASCAN_IMAGE} scan -t aws -i terraform -d terraform/aws > terrascan_report.txt || true
                
	                echo "--- CONTENIDO DEL REPORTE ---"
	                cat terrascan_report.txt
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
