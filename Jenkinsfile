def genericsh(cmd){
    if(isUnix()){
        sh cmd
    }else{
        bat cmd
    }
}

pipeline {
    agent any //Con agent none hay que especificar el agente en los stages, sino agent any

    stages {
        stage('Branch') {
            agent { label 'linux' }
            steps {
               echo env.BRANCH_NAME
            }
        }
        stage('Build') {
            steps {
               echo 'Hacemos como que compilamos en python...'
               echo WORKSPACE
               genericsh 'ls'
            }
        }
        stage('Test'){
            parallel{
                stage('Unit') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            echo WORKSPACE
                            genericsh'''
                                export PYTHONPATH=.
                                pytest --junitxml=result-unit.xml test//unit
                            '''
                        }
                    }
                }
                stage('Rest') {
                    steps {
                        catchError(buildResult: 'UNSTABLE', stageResult: 'FAILURE'){
                            echo WORKSPACE
                            genericsh'''
                                export FLASK_APP=app//api.py
                                flask run &
                                java -jar //home//jenkins//agent//wiremock//wiremock-jre8-standalone-2.28.0.jar --port 9090 --root-dir test//wiremock --verbose &
                                sleep 1
                                export PYTHONPATH=.
                                pytest --junitxml=result-test.xml test//rest
                            '''
                        }
                    }
                }
            }
        }
        stage('Deploy') {
            agent { label 'deploy' }
            when {
                beforeAgent true;
                expression {
                    return env.BRANCH_NAME == 'main';
                }
            }
            steps {
                echo WORKSPACE
                echo 'Solo funciono en la rama de main'
            }
        }
        stage('Results') {
            steps {
                junit 'result-unit.xml'
            }
        }
    }
}