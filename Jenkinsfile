pipeline {
    agent any

    stages {

        stage('Echo inicial') {
            steps {
                echo 'Iniciando pipeline para caso practico 1.1'
            }
        }

        stage('Verificar descarga del codigo') {
            steps {
                bat 'dir'
            }
        }

        stage('Verificar workspace') {
            steps {
                bat 'echo Workspace actual: %WORKSPACE%'
            }
        }

        stage('Build') {
            steps {
                echo 'Etapa Build (Python - sin acciones reales)'
            }
        }

        stage('Tests') {
            parallel {

                stage('Unit') {
                    steps {
                        bat '''
                        SET PYTHONPATH=%WORKSPACE%
                        pytest --junitxml=result-unit.xml test\\unit
                        '''
                    }
                }

                stage('Service') {
                    steps {
                        bat '''
                        SET FLASK_APP=app\\api.py
                        start flask run
                        start java -jar C:\\Users\\jorge.unir\\Downloads\\wiremock-standalone-3.13.2.jar --port 9090 --root-dir "%WORKSPACE%\\test\\wiremock"
                        SET PYTHONPATH=%WORKSPACE%
                        pytest --junitxml=result-service.xml test\\rest
                        '''
                    }
                }
            }
        }

        stage('Result') {
            steps {
                junit 'result-*.xml'
            }
        }
    }
}
