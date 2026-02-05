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

        stage('Scan IaC - Terrascan JSON') {
            steps {
                script {
                    def containerName = "terrascan-${BUILD_NUMBER}"
                    def jsonFile = "terrascan_results.json"
                    
                    // Comandos concatenados:
                    // 1. Ejecuta el scan
                    // 2. -o json: Formato JSON
                    // 3. > /data/...: Guarda el resultado en un archivo DENTRO del contenedor
                    // 4. || true: Evita que Jenkins marque error si Terrascan encuentra vulnerabilidades (queremos el reporte igual)
                    def scanCmd = "/go/bin/terrascan scan -i terraform -t aws -d /data/terraform/aws -o json > /data/${jsonFile} || true"

                    try {
                        echo "--- Configurando Contenedor ---"
                        // Creamos el contenedor preparado para ejecutar el comando y guardar el archivo
                        sh "docker create --name ${containerName} --entrypoint /bin/sh ${TERRASCAN_IMAGE} -c '${scanCmd}'"
                        
                        echo "--- Copiando Código Fuente al Contenedor ---"
                        sh "docker cp . ${containerName}:/data"
                        
                        echo "--- Ejecutando Escaneo y Generando JSON ---"
                        sh "docker start -a ${containerName}"
                        
                        echo "--- Extrayendo Reporte JSON hacia Jenkins ---"
                        // COPIAR DESDE EL CONTENEDOR HACIA EL WORKSPACE
                        sh "docker cp ${containerName}:/data/${jsonFile} ./${jsonFile}"
                        
                        // Opcional: Imprimir el JSON en la consola también para verlo rápido
                        sh "cat ${jsonFile}"
                        
                    } finally {
                        sh "docker rm -f ${containerName} || true"
                    }
                    
                    // Este paso 'guarda' el archivo en la interfaz de Jenkins para que puedas descargarlo
                    archiveArtifacts artifacts: jsonFile, allowEmptyArchive: true
                }
            }
        }
    }
}
