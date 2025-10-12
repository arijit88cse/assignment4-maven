/**
 * Sanders & Fresco CI/CD Pipeline for Maven WebApp (Optimized for OpenJDK 21 Headless JRE Runtime)
 */
pipeline {
    agent any

    // Environment variables for centralizing configuration
    environment {
        // Tomcat Deployment
        TOMCAT_URL = 'http://172.31.45.21:9080/manager/text'
        TOMCAT_CREDENTIALS_ID = 'tomcat-creds'
        APP_CONTEXT_PATH = '/assignment4-maven'

        // SonarQube Configuration
        SONAR_HOST_URL = 'http://172.31.45.21:9000'
        SONAR_SERVER_NAME = 'SonarQube Server'
        
        MAVEN_OPTS = "-Dmaven.test.failure.ignore=true"
    }

    // Define the tool names configured in Manage Jenkins -> Tools
    tools {
        // Use the full JDK for compilation and testing
        jdk 'JDK-21' 
        maven 'M3' 
    }

    stages {
        // 1. JIRA JOB: COMPILE 
        stage('Compile & Prepare') {
            steps {
                echo 'Stage 1: Compiling source code and preparing build environment using JDK-21...'
                sh 'mvn clean verify -DskipTests=true'
            }
        }

        // 2. JIRA JOB: CODE REVIEW (SonarQube)
        stage('Code Review (Sonar)') {
            steps {
                echo 'Stage 2: Starting SonarQube Code Analysis on port 9000...'
                withSonarQubeEnv(SONAR_SERVER_NAME) {
                    sh "mvn sonar:sonar -Dsonar.projectKey=${project.artifactId} -Dsonar.host.url=${SONAR_HOST_URL}"
                }
            }
        }
        
        // 3. JIRA JOB: UNIT TEST
        stage('Unit Test') {
            steps {
                echo 'Stage 3: Running Unit Tests...'
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }

        // 4. JIRA JOB: PACKAGE (Creates the WAR file)
        stage('Package WAR') {
            steps {
                echo 'Stage 4: Packaging the application into a WAR file...'
                sh 'mvn package -DskipTests'
            }
            post {
                success {
                    archiveArtifacts artifacts: 'target/*.war', fingerprint: true
                }
            }
        }

        // 5. JIRA JOB: DEPLOY
        stage('Deploy to Tomcat') {
            steps {
                echo "Stage 5: Deploying WAR to Tomcat Manager on http://172.31.45.21:9080/..."
                
                // Securely fetch credentials from Jenkins Credential Store
                withCredentials([usernamePassword(credentialsId: "${TOMCAT_CREDENTIALS_ID}", 
                                                   passwordVariable: 'TOMCAT_PASSWORD', 
                                                   usernameVariable: 'TOMCAT_USERNAME')]) {
                    
                    // Deploy command using the Tomcat Maven Plugin
                    // The Tomcat server itself uses the OpenJDK 21 Headless JRE to run the WAR.
                    sh "mvn org.apache.tomcat.maven:tomcat7-maven-plugin:2.2:deploy " +
                       "-Durl=${TOMCAT_URL} " +
                       "-Dpath=${APP_CONTEXT_PATH} " +
                       "-Dusername=${TOMCAT_USERNAME} " +
                       "-Dpassword=${TOMCAT_PASSWORD}"
                }
                echo "Deployment complete. Application available at: http://172.31.45.21:9080${APP_CONTEXT_PATH}"
            }
        }
    }
}
