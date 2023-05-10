pipeline {

    agent any

    environment {
      registry = "yasser74/vproapp"
      registrtCredentials = 'yasser-dockerhub'
    }

    stages{


        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'sed -i 's/0.7.2.201409121644/0.8.5/g' pom.xml '
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

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'sonar-pro'
            }

            steps {
                withSonarQubeEnv('sonar-vprofile') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage("Build App Image") {
            steps{
                scipt{
                  dockerImage = docker.build registry + ":v$BUILD_NUMBER"
                }
            }

            
        }
        stage("Push App Image") {
            steps{
                scipt{
                  docker.withRegistry ('',registrtCredentials){
                    dockerImage.push("v$BUILD_NUMBER")
                  }
            }

            
        }

        stage("Remove unused Images") {
            steps{
                sh "docker rmi $registry:v$BUILD_NUMBER"
            }
        }

        stage("Kubernetes deploy ") {
            agent EKS
            steps{
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage= ${registry}:v${BUILD_NUMBER} --namespace prod1"
            }
        }


    }


}
