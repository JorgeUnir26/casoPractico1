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
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat 'dir'
                }
            }
        }

        stage('Instalando Dependencias') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                        python -m pip install --upgrade pip
                        pip install pytest pytest-cov flake8 bandit
                    '''
                }
            }
        }

        stage('Unit') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                        SET PYTHONPATH=%WORKSPACE%
                        pytest --junitxml=result-unit.xml --cov=. --cov-report=xml test\\unit
                    '''
                    stash name: 'test-results', includes: 'result-unit.xml, coverage.xml', allowEmpty: true
                }
            }
        }

        stage('Rest') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
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

        stage('Flake8') {
            steps {
               catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
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
                            echo "Build UNHEALTHY: ${issues} hallazgos (>=10)"
                        } 
                        else if (issues >= 8) {
                            echo "Build UNSTABLE: ${issues} hallazgos (>=8)"
                        } 
                        else {
                            echo "Calidad aceptable (${issues} hallazgos)"
                        }
                    }
                }
            }
        }

        stage('Security Test') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
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
                            echo "Build UNHEALTHY: ${issues} problemas de seguridad"
                        }
                        else if (issues >= 2) {
                            echo "Build UNSTABLE: ${issues} problemas de seguridad"
                        }
                        else {
                            echo "Seguridad aceptable (${issues} problemas)"
                        }
                    }
                }
            }
        }

        stage('Coverage') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    unstash 'test-results'
                    bat 'coverage report || echo "No coverage data found"'
                }
            }
        }

        stage('Performance') {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                    bat '''
                        "C:\\Users\\jorge.unir\\Downloads\\apache-jmeter-5.6.3\\apache-jmeter-5.6.3\\bin\\jmeter.bat" -n -t test\\jmeter\\performance_test.jmx -l test\\jmeter\\results.jtl -j test\\jmeter\\jmeter.log
                    '''
                }
            }
        }

    }
}
