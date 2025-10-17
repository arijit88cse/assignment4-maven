pipeline {
    // Agent defines where the pipeline will run (e.g., on the 'master' or a specific 'maven' label)
    agent any

    // Tools setup (ensure 'Maven-3.6.3' is configured in Global Tool Configuration)
    tools {
        maven 'Maven-3.6.3'
    }

    stages {
        // --------------------------------------------------------------------------------
        stage('Checkout') {
            steps {
                echo 'Checking out source code...'
                // Replace with your actual SCM configuration (e.g., git checkout)
                // git url: 'your-repository-url.git', branch: 'main'
            }
        }
        // --------------------------------------------------------------------------------
        stage('Compile') {
            steps {
                echo 'Compiling the project...'
                // Maven compile goal: compiles the source code
                sh 'mvn clean compile'
            }
        }
        // --------------------------------------------------------------------------------
        stage('Code Review') {
            steps {
                echo 'Running SonarQube analysis for code review...'
                // Assumes SonarQube is configured as a build wrapper or a separate step
                // sh 'mvn sonar:sonar'
                
                // For this example, we'll use a placeholder step.
                script {
                    if (env.SKIP_CODE_REVIEW == 'true') {
                        echo 'Skipping code review stage.'
                    } else {
                        echo 'Code review finished. Quality Gate passed (simulated).'
                    }
                }
            }
        }
        // --------------------------------------------------------------------------------
        stage('Unit Test') {
            steps {
                echo 'Running unit tests...'
                // Maven test goal: runs unit tests and skips packaging
                sh 'mvn test'
                
                // Post-processing step to publish test results (e.g., JUnit reports)
                // This step ensures the test results are visible in Jenkins
                junit '**/target/surefire-reports/*.xml'
            }
        }
        // --------------------------------------------------------------------------------
        stage('Package') {
            steps {
                echo 'Creating the application package (JAR/WAR)...'
                // Maven package goal: compiles, runs tests, and packages the application
                sh 'mvn package -DskipTests' 
                // Note: Tests are run in the 'Unit Test' stage, so they are skipped here.
            }
            post {
                // Archive the generated artifact
                always {
                    archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                }
            }
        }
        // --------------------------------------------------------------------------------
        stage('Deploy') {
            steps {
                echo 'Deploying the artifact to a server (e.g., Tomcat, Nexus, Kubernetes)...'
                // This is a placeholder. Real deployment requires specific tooling/scripts.
                // sh 'scp target/your-app.jar user@server:/app/deploy' 
                echo 'Deployment successful (simulated).'
            }
        }
		// --------------------------------------------------------------------------------
        stage('Package') {
            steps {
                echo 'Creating the application package (WAR)...'
                // Runs build, creates WAR file in target/
                sh 'mvn package -DskipTests' 
            }
            post {
                always {
                    // Archive the generated artifact for visibility/download
                    archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
                }
            }
        }
        // --------------------------------------------------------------------------------
        stage('Deploy & Restart Tomcat') {
            // Define variables for clarity (adjust if your WAR filename is different)
            environment {
                TOMCAT_HOME = '/tomcat/apache-tomcat'
                WAR_FILE = 'target/*.war' // Assumes a single WAR file is produced
                APP_NAME = 'myapp.war'    // Target name for deployment
            }
            steps {
                echo 'Stopping Tomcat server...'
                // 1. STOP Tomcat
                // Use shutdown.sh script in the specified bin path.
                // Redirect output to /dev/null to handle case where server is already stopped (optional)
                sh "sh \$TOMCAT_HOME/bin/shutdown.sh"
                // Give Tomcat a few seconds to shut down gracefully
                sh 'sleep 5' 
                
                echo 'Deploying artifact to webapps folder...'
                // 2. MOVE/COPY the WAR file
                // Find the built WAR file and copy it to the tomcat webapps directory
                // We rename it to the desired application name (e.g., ROOT.war or myapp.war)
                sh "cp \${WAR_FILE} \${TOMCAT_HOME}/webapps/\${APP_NAME}"
                
                echo 'Starting Tomcat server on port 9080...'
                // 3. START Tomcat
                // Use startup.sh script in the specified bin path.
                sh "sh \$TOMCAT_HOME/bin/startup.sh"
                
                // Add a check or a short sleep to wait for startup (optional but recommended)
                sh 'sleep 10' 

                echo "Deployment successful. Application should be available at port 9080."
            }
        }
        // --------------------------------------------------------------------------------
    }
}
