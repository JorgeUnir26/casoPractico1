pipeline {
    agent any

    stages {

        stage('Iniciando Caso Practico 1.2 | Reto 1 - Pipeline CI') {
            steps {
                echo 'Iniciando pipeline para caso practico 1.2'
            }
        }

        stage('Ver workspace') {
            steps {
                bat 'dir'
            }
        }

        stage('Instalando Dependencias') {
            steps {
                bat '''
                    python -m pip install --upgrade pip
                    pip install pytest pytest-cov flake8 bandit
                '''
            }
        }

        stage('Unit') {
            steps {
                bat '''
                    SET PYTHONPATH=%WORKSPACE%
                    pytest ^
                      --junitxml=result-unit.xml ^
                      --cov=. ^
                      --cov-report=xml ^
                      test\\unit
                '''

                stash name: 'test-results', includes: 'result-unit.xml, coverage.xml'
            }
        }

        stage('Rest') {
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

        stage('Flake8') {
            steps {
               script {
                    bat '''
                        python -m flake8 . > flake8_report.txt || exit 0
                    '''

                    def issues = readFile('flake8_report.txt')
                                    .readLines()
                                    .findAll { it.trim() }
                                    .size()

                    echo "Hallazgos encontrados flake8: ${issues}"

                    if (issues >= 10) {
                        error("Build UNHEALTHY: ${issues} hallazgos (>=10)")
                    } 
                    else if (issues >= 8) {
                        unstable("Build UNSTABLE: ${issues} hallazgos (>=8)")
                    } 
                    else {
                        echo "Calidad aceptable (${issues} hallazgos)"
                    }
                }
            }
        }

        stage('Security Test') {
            steps {
                script {
                    bat '''
                        python -m bandit -r . -f txt > bandit_report.txt || exit 0
                    '''

                    def issues = readFile('bandit_report.txt')
                                    .readLines()
                                    .findAll { it.startsWith('>> Issue') }
                                    .size()

                    echo "Hallazgos de seguridad (Bandit): ${issues}"

                    if (issues >= 4) {
                        error("Build UNHEALTHY: ${issues} problemas de seguridad")
                    }
                    else if (issues >= 2) {
                        unstable("Build UNSTABLE: ${issues} problemas de seguridad")
                    }
                    else {
                        echo "Seguridad aceptable (${issues} problemas)"
                    }
                }
            }
        }

        stage('Coverage') {
            steps {
                unstash 'test-results'

                bat '''
                    coverage report
                '''
            }
        }

        stage('Performance') {
            steps {
                bat '''
                    "C:\\Users\\jorge.unir\\Downloads\\apache-jmeter-5.6.3\\apache-jmeter-5.6.3\\bin\\jmeter.bat" -n -t test\\jmeter\\performance_test.jmx -l test\\jmeter\\results.jtl -j test\\jmeter\\jmeter.log
                '''
            }
        }

    }
}
