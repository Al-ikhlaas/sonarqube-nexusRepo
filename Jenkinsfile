pipeline {
    agent any
    environment {
        MAVEN_OPTS = "--add-opens java.base/java.lang=ALL-UNNAMED"
    }

    stages {
        stage('Build with maven') {
            steps {
                sh 'cd SampleWebApp && mvn clean install'
            }
        }

        stage('Test') {
            steps {
                sh 'cd SampleWebApp && mvn test'
            }
        }

        stage('Code Quality Scan') {
            steps {
                withSonarQubeEnv('sonar-scanner') {
                    sh "mvn -f SampleWebApp/pom.xml sonar:sonar"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: true
            }
        }

        stage('Push to Nexus') {
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'SampleWebApp', classifier: '', file: 'SampleWebApp/target/SampleWebApp.war', type: 'war']], 
                                      credentialsId: 'nexuspass', 
                                      groupId: 'SampleWebApp', 
                                      nexusUrl: 'ec2-54-84-162-136.compute-1.amazonaws.com:8081/', 
                                      nexusVersion: 'nexus3', 
                                      protocol: 'http', 
                                      repository: 'maven-snapshots', 
                                      version: '1.0-SNAPSHOT'
            }
        }

        stage('Deploy to Tomcat') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'tompass', path: '', url: 'http://3.81.142.8:8080/')], 
                       contextPath: 'myapp', 
                       war: '**/*.war'
            }
        }
    }
}
