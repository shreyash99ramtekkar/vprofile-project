pipeline {
    agent any
    
    tools {
        jdk "OracleJDK17"    
        maven "Maven 3.9.10"
    }
    
    environment {
        SNAP_REPO = "vprofile-snapshot"
        NEXUS_USER = "devops"
        NEXUS_PASS = "devops123"
        RELEASE_REPO = "vprofile-release"
        CENTRAL_REPO = "vprofile-maven-central"
        NEXUS_GRP_REPO = "vpro-maven-group"
        NEXUSIP = "192.168.57.13"
        NEXUSPORT = "8081"
        SONARSERVER = "SonarServer"
        SONARSCANNER = "SonarScanner"
        NEXUS_PROTOCOL = "http"
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
         stage('SonarQube analysis') {
            environment {
                scannerHome = tool("${SONARSCANNER}")
            }
            steps{
                withSonarQubeEnv("SonarScanner") {
                    sh '''${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=vprofile \
                        -Dsonar.projectName=Vprofile \
                        -Dsonar.projectVersion=1.0 \
                        -Dsonar.sources=src/ \
                        -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                        -Dsonar.junit.reportPaths=target/surefire-reports/ \
                        -Dsonar.jacoco.reportPaths=target/jacoco.exec \
                        -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml
                    '''
                }
            }
        }
  }
        stage("Test"){
            steps{
                sh 'mvn test'
            }
        }
        stage("CheckStyle Analytsis"){
                steps {
                    sh 'mvn checkstyle:checkstyle'
                }
        }
    }
}
