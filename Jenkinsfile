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
                    echo "--- 🕵️‍♂️ Verificando Archivos en Jenkins ---"
                    // Tu grep estaba bien, confirmamos que el archivo existe en el workspace
                    sh "grep -C 5 'web_host_storage' terraform/aws/ec2.tf"
                    
                    echo "--- Preparando Contenedor Terrascan ---"
                    // 1. Definimos un nombre único para el contenedor
                    def containerName = "terrascan-${BUILD_NUMBER}"
                    
                    try {
                        // 2. Creamos el contenedor (sin arrancarlo aún)
                        sh "docker create --name ${containerName} --entrypoint /bin/sh ${TERRASCAN_IMAGE}"
                        
                        // 3. COPIAMOS los archivos del Workspace de Jenkins AL contenedor
                        // Esto evita el problema de los volúmenes en Docker-in-Docker
                        sh "docker cp . ${containerName}:/data"
                        
                        echo "--- Ejecutando Escaneo ---"
                        // 4. Ejecutamos el comando dentro del contenedor ya cargado con los archivos
                        // Nota: Usamos 'docker start -a' para ver la salida (attach)
                        sh """
                            docker start -a ${containerName} \
                            /go/bin/terrascan scan \
                            -i terraform \
                            -t aws \
                            -d /data/terraform/aws \
                            --verbose || true
                        """
                    } finally {
                        // 5. Limpieza: Borramos el contenedor siempre
                        sh "docker rm -f ${containerName} || true"
                    }
                }
            }
        }
    }
}
