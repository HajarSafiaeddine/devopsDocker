pipeline{
    agent any
    tools{
        maven "MAVEN3"
        jdk "JDK8"
    }
    stages{
        stage('fetch the code'){
          steps{
            git branch: 'vp-rem', url:'https://github.com/devopshydclub/vprofile-repo.git'
          }
        }
        stage('build the code'){
            steps{
                sh 'mvn install -Dskiptests'
            post{
                success{
                    echo('Succes! Now we are archiving the artefact')
                    archiveArtifacts artifacts: '**/*.war'
                }
            }
            }
        }
        stage("test the code"){
           steps{
            sh 'mvn test'
           }
        }
        stage("code analysis with checkstyle"){
           steps{
            sh 'mvn checkstyle:checkstyle'
           }
        }
        stage('code analysis with Sonarqube'){
            environment{
                scannerHome = tool 'Sonar';
            }
            steps{
              withSonarQubeEnv('SonarServer') { 
                   sh '''${scannerHome}/bin/sonar-scanner \
         -Dsonar.projectKey=vproapp \
         -Dsonar.projectName=vproapp \
         -Dsonar.sources=src/ \
         -Dsonar.junit.reportsPath=target/surefire-reports \
         -Dsonar.jacoco.reportPath=target/jacoco.exec \
         -Dsonar.java.binaries=src/test/ \
         -Dsonar.projectVersion=${BUILD_ID}'''
        }
            }
        }
        stage('code analysis with sonar quality gates') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }
       stage('upload the artifact to Nexus repo') {
        steps {
      nexusArtifactUploader(
    nexusVersion: 'nexus3',
    protocol: 'http',
    nexusUrl: '34.229.73.203:8081',
    groupId: 'com.example',
    version: "${BUILD_ID}",
    repository: 'vproapp',
    credentialsId: 'Nexus',
    artifacts: [
        [artifactId: 'vproapp',
         classifier: '',
         file: 'target/vprofile-v2.war',
         type: 'war']
    ]
 )
    }
}
    }
}