pipeline {
    agent any
    
    tools {
        jdk "OracleJDK17"    
        maven "Maven 3.9.10"
    }
    
    environment {
        SNAP_REPO = "vprofile-snapshot"
        NEXUS_USER = "devops"
        NEXUS_PASS = "qwerty123"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vprofile-maven-central"
        NEXUS_GRP_REPO = "vpro-maven-group"
        NEXUSIP = "192.168.57.13"
        NEXUSPORT = "8081"
        NEXUSLOGIN = "nexuslogin"
        SONARSERVER = "SonarServer"
        SONARSCANNER = "SonarScanner"
        NEXUS_PROTOCOL = "http"
        PROJECT_NAME = "vprofile"
    }
    stages{
        stage('BUILD'){
            steps {
                sh 'mvn -s settings.xml clean install -DskipTests'
            }
            post{
                success {
                    echo("Now Archive the build artifacts")
                    archiveArtifacts artifacts: '**/target/*.war'
                }
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
        stage('SonarQube analysis') {
            environment {
                scannerHome = tool("${SONARSCANNER}")
            }
            steps{
                withSonarQubeEnv("${SONARSERVER}") {
                    sh """${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${PROJECT_NAME} \
                        -Dsonar.projectName=${PROJECT_NAME} \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportPaths=target/surefire-reports/ \
                        -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    """
                }
            }
        }
        stage("Quality Gate"){
            steps{
                timeout(time: 1, unit: 'HOURS') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage("Deploy to Nexus Snapshot"){
            steps{
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                    groupId: 'com.vprofile.qa',
                    version: "${env.BUILD_NUMBER}-${env.BUILD_TIMESTAMP}",
                    repository: "${RELEASE_REPO}",
                    credentialsId: "${NEXUSLOGIN}",
                    artifacts: [
                        [artifactId: ${PROJECT_NAME},
                        classifier: '',
                        file: "target/${PROJECT_NAME}-v2.war",
                        type: 'war']
                    ]
                )
            }
        }
    
        stage("Test"){
            steps{
                sh 'mvn test'
            }
        }
        
    }
}

