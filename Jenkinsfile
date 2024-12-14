pipeline {
    agent any  // Esto permite ejecutar el pipeline en cualquier nodo disponible

    environment {
        PROJECT_DIR = 'C:/Users/Krewin/Desktop/Jetkins'  // Carpeta local donde está el proyecto
        BACKUP_DIR = 'C:/Users/Krewin/Desktop/Jetkins/backup'  // Directorio de respaldo
        PROD_DIR = '/ruta/produccion'  // Ruta en el servidor de producción
        TEST_SERVER = 'user@servidor-pruebas'  // Acceso al servidor de pruebas
        PROD_SERVER = 'user@servidor-produccion'  // Acceso al servidor de producción
    }

    stages {
        // 1. Checkout del código
        stage('Checkout') {
            steps {
                git branch: 'desarrollo', url: 'https://github.com/Krewin9/JETKINS_PROJECT.git'
            }
        }

        // 2. Fusión en la rama de pruebas
        stage('Merge to Pruebas') {
            steps {
                script {
                    // Cambiar a la rama de pruebas
                    sh "git checkout pruebas"
                    // Hacer merge de la rama desarrollo
                    sh "git merge desarrollo"
                    // Push de los cambios a la rama pruebas
                    sh "git push origin pruebas"
                }
            }
        }

        // 3. Desplegar en servidor de pruebas
        stage('Deploy to Test Server') {
            steps {
                script {
                    // Copiar los archivos al servidor de pruebas
                    sh "scp -r ${PROJECT_DIR}/* ${TEST_SERVER}:/ruta-del-servidor-pruebas"
                }
            }
        }

        // 4. Fusión en la rama de producción
        stage('Merge to Production') {
            steps {
                script {
                    // Cambiar a la rama de producción
                    sh "git checkout producción"
                    // Hacer merge de la rama pruebas
                    sh "git merge pruebas"
                    // Push de los cambios a la rama producción
                    sh "git push origin producción"
                }
            }
        }

        // 5. Copiar al directorio de producción
        stage('Deploy to Production') {
            steps {
                script {
                    // Crear un respaldo del directorio de producción
                    sh "rsync -avz ${PROD_SERVER}:${PROD_DIR} ${BACKUP_DIR}"
                    // Copiar los archivos al directorio de producción
                    sh "scp -r ${PROJECT_DIR}/* ${PROD_SERVER}:${PROD_DIR}"
                }
            }
        }

        // 6. Restaurar en caso de falla
        stage('Restore Production if Needed') {
            steps {
                input message: '¿Restaurar respaldo?', ok: 'Sí'
                script {
                    // Restaurar los archivos desde el respaldo si se necesita
                    sh "rsync -avz ${BACKUP_DIR}/* ${PROD_SERVER}:${PROD_DIR}"
                }
            }
        }
    }

    post {
        always {
            
            sh "echo 'Pipeline completado' >> ${PROJECT_DIR}/pipeline.log"
        }

        success {
            echo 'Pipeline ejecutado exitosamente'
        }

        failure {
            echo 'Pipeline falló. Se necesita revisar.'
        }
    }
}
