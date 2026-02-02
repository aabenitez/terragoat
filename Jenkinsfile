pipeline {
    agent any
    environment {
        TERRASCAN_IMAGE = 'tenable/terrascan:latest'
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/aabenitez/terragoat.git', branch: 'master'
            }
        }

        stage('Security Scan') {
            steps {
                script {
                    echo "Iniciando escaneo de infraestructura en terraform/aws..."
                    // Eliminamos el flag -u para evitar líos de permisos y montamos la subcarpeta directo a /iac
                    sh """
			docker run --rm \
			    -v ${WORKSPACE}/terraform/aws:/iac \
			    -w /iac \
			    ${TERRASCAN_IMAGE} scan -i terraform -t aws --verbose || true
                    """
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'terrascan_report.txt', fingerprint: true
            echo "Limpiando imagen Docker..."
            sh "docker rmi ${TERRASCAN_IMAGE} || true"
            deleteDir()
        }
    }
}
