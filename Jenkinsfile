pipeline {
    agent any

    environment {
        // Imagen oficial de Terrascan
        TERRASCAN_IMAGE = 'tenable/terrascan:latest'
        
        // Argumentos para Docker:
        // -rm: Borra el contenedor al terminar
        // -v ${WORKSPACE}:/data: Monta la carpeta de Jenkins dentro del contenedor en /data
        // -w /data: Establece /data como directorio de trabajo
        DOCKER_ARGS = '--rm -v ${WORKSPACE}:/data -w /data'
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
                    doGenerateSubmoduleConfigurations: false,
                    extensions: [[$class: 'CloneOption', depth: 0, noTags: false, reference: '', shallow: false]],
                    userRemoteConfigs: [[url: 'https://github.com/pablotpy/terragoat-fix']] 
                ])
            }
        }

        stage('Scan IaC - Terrascan') {
            steps {
                script {
                    echo "--- 🕵️‍♂️ TESTIGO DE JENKINS: ec2.tf ---"
                    // Esto imprimirá lo que REALMENTE tiene Jenkins en su disco
                    sh "grep -C 5 'web_host_storage' terraform/aws/ec2.tf"
                    
                    echo "--- Iniciando Escaneo ---"
                    sh """
                        docker run ${DOCKER_ARGS} ${TERRASCAN_IMAGE} scan \
                        -i terraform \
                        -t aws \
                        -d terraform/aws \
                        --verbose || true
                    """
                }
            }
        }
    }
}
