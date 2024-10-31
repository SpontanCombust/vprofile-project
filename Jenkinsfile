pipeline {
    agent any

    tools {
        maven "maven3"
    }

    environment {
        ECR_REGISTRY = credentials('ecr-registry')
        ECR_REPOSITORY = credentials('ecr-repository')
        DOCKER_IMAGE = '$ECR_REGISTRY/$ECR_REPOSITORY:V$BUILD_NUMBER'
    }

    stages{
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: 'target/vprofile-v2.war'
                }
            }
        }
        
        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        // stage('CODE ANALYSIS with SONARQUBE') {
        //     environment {
        //         scannerHome = tool 'mysonarscanner4'
        //     }

        //     steps {
        //         withSonarQubeEnv('sonar-pro') {
        //             sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
        //            -Dsonar.projectName=vprofile-repo \
        //            -Dsonar.projectVersion=1.0 \
        //            -Dsonar.sources=src/ \
        //            -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
        //            -Dsonar.junit.reportsPath=target/surefire-reports/ \
        //            -Dsonar.jacoco.reportsPath=target/jacoco.exec \
        //            -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
        //         }

        //         timeout(time: 10, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true
        //         }
        //     }
        // }

        stage('Build Docker App image') {
            steps {
                script {
                    dockerImage = docker.build "$DOCKER_IMAGE"
                }
            }
        }

        stage('Upload image') {
            steps {
                script {
                    docker.withRegistry('$ECR_REGISTRY', 'ecr:us-east-1:aws-creds') {
                        dockerImage.push('V$BUILD_NUMBER')
                        dockerImage.push("latest")
                    }
                }
            }
        }

        stage("Remove unused Docker images") {
            steps {
                sh 'docker rmi $DOCKER_IMAGE'
            }
        }

        // stage("Deploy to kubernetes cluster") {
        //     agent {
        //         label 'kops'
        //     }

        //     steps {
        //         sh 'helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=$DOCKER_IMAGE --namespace prod'
        //     }
        // }
    }
}
