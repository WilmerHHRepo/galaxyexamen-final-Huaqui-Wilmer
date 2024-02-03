pipeline {
    agent any
    environment {
        DOCKER_CREDS = credentials('docker-credentials')
        }
        stages {
            stage('Build') {
                agent {
                    docker { image 'maven:3.6.3-openjdk-11-slim' }
                }
                    steps {
                        //sh 'gradle build'
                        sh 'mvn -B verify'
                        //archiveArtifacts artifacts: 'build/libs/labgradle-*-SNAPSHOT.jar', fingerprint: true
                        //archiveArtifacts artifacts: 'build/libs/*-SNAPSHOT.jar', fingerprint: true
                        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true, onlyIfSuccessful: true
                        sh 'echo carpetas'
                        sh 'ls -l target/classes/'
                    }
            }
/*            stage('Test') {
                agent {
                    docker { image 'gradle:7.5.1-jdk11' }
                }
                    steps {
                        sh 'gradle test'
                        junit 'build/test-results/test/TEST-*.xml'
                    }
            }*/
            stage('SonarQube') {
                steps {
                    script{
                        def scannerHome = tool 'scanner-default'
                        withSonarQubeEnv('sonar-server') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=labmaven \
                            -Dsonar.projectName=labmaven \
                            -Dsonar.sources=src/main \
                            -Dsonar.sourceEncoding=UTF-8 \
                            -Dsonar.language=java \
                            -Dsonar.tests=src/test \
                            -Dsonar.junit.reportsPath=target/surefire-reports \
                            -Dsonar.surefire.reportsPath=target/surefire-reports \
                            -Dsonar.jacoco.reportPath=target/jacoco.exec \
                            -Dsonar.java.binaries=target/classes \
                            -Dsonar.java.coveragePlugin=jacoco \
                            -Dsonar.coverage.jacoco.xmlReportPaths=target/jacoco.xml \
                            -Dsonar.exclusions=**/*IT.java,**/*TEST.java,**/*Test.java,**/src/it**,**/src/test**,**/gradle/wrapper** \
                            -Dsonar.java.libraries=target/*.jar"
                        }
                    }
                }
            }
            stage('Build Image') {
                steps {
                    copyArtifacts filter: 'build/libs/labgradle-*-SNAPSHOT.jar',
                                    fingerprintArtifacts: true,
                                    projectName: '${JOB_NAME}',
                                    flatten: true,
                                    selector: specific('${BUILD_NUMBER}'),
                                    target: 'build/libs/'
                    sh 'docker --version'
                    sh 'docker-compose --version'
                    sh 'docker-compose build'
                }
            }
            stage('Publish Image') {
                steps {
                    script {
                        sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                        sh 'docker tag msmicroservice ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker push ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker logout'
                    }
                }
            }
            stage('Run Container') {
                steps {
                    script {
                        sh 'docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}'
                        sh 'docker rm galaxyLab -f'
                        sh 'docker run -d -p 8080:8080 --name galaxyLab ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        //sh 'docker run -d -p 8080:8080 ${DOCKER_CREDS_USR}/msmicroservice:$BUILD_NUMBER'
                        sh 'docker logout'
                    }
                }
            }
            stage('Test Run Container') {
                    steps {
                        script {
                            sh 'docker ps'
                        //    sh 'curl http://192.168.1.17:8080/customers'
                        }
                    }
            }
        }
}
