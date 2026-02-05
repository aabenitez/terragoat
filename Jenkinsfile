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
                    userRemoteConfigs: [[url: 'https://github.com/aabenitez/terragoat']] 
                ])
            }
        }

        stage('Scan IaC - Terrascan') {
            steps {
                script {
                    echo "--- 🕵️‍♂️ Verificando Archivos en Jenkins ---"
                    sh "grep -C 5 'web_host_storage' terraform/aws/ec2.tf"
                    
                    def containerName = "terrascan-${BUILD_NUMBER}"
                    // Definimos el comando EXACTO que queremos correr dentro del contenedor
                    def scanCmd = "/go/bin/terrascan scan -i terraform -t aws -d /data/terraform/aws --verbose"
                    
                    try {
                        echo "--- Creando Contenedor con el comando preparado ---"
                        // 1. CREATE: Pasamos el comando aquí usando 'sh -c'
                        // Esto le dice al contenedor: "Cuando arranques, ejecuta esto"
                        sh "docker create --name ${containerName} --entrypoint /bin/sh ${TERRASCAN_IMAGE} -c '${scanCmd}'"
                        
                        echo "--- Copiando Archivos ---"
                        // 2. CP: Copiamos los archivos
                        sh "docker cp . ${containerName}:/data"
                        
                        echo "--- Ejecutando Escaneo ---"
                        // 3. START: Solo arrancamos (el comando ya está inyectado desde el paso 1)
                        // '-a' es para adjuntar la salida (attach) y ver los logs en Jenkins
                        sh "docker start -a ${containerName}"
                        
                    } finally {
                        sh "docker rm -f ${containerName} || true"
                    }
                }
            }
        }
    }
}
