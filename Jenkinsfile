pipeline {
    agent any
    
    tools {
        jdk 'jdk17'
        maven 'maven3'
    }
    environment{
        SCANNER_HOME = tool 'sonar-scanner'
    }
    
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/DARR3NG/Mission.git'
            }
        }
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }
        
        stage('Test') {
            steps {
                sh "mvn test -DskipTests=true"
            }
        }
        
        stage('Trivy Scan FS') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                
                withSonarQubeEnv('sonar') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner  -Dsonar.projectName=Mission  -Dsonar.projectKey=Mission  \
                    -Dsonar.java.binaries=.  '''
    
                }
            }
        }
        stage('Build') {
            steps {
                sh "mvn package -DskipTests=true"
            }
        }
        stage('Deploy Artifact To Nexus') {
            steps {
                
                withMaven(globalMavenSettingsConfig: 'maven-setting', jdk: 'jdk17', maven: 'maven3', mavenSettingsConfig: '', traceability: true) {

                    sh "mvn deploy -DskipTests=true"
                }
            }
        }
                
        stage('Build & Tag Docker Image') {
            steps {
                
                script{ 
                // This step should not normally be used in your script. Consult the inline help for details.
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    
                    sh "docker build -t darr3ng/mission:latest ."

                }
                    
                }
            }
        }
        
        
                
        stage('Trivy Scan Image') {
            steps {
                sh "trivy image --format table -o trivy-fs-report.html darr3ng/mission:latest"
            }
        }
        
        
        stage('Publish Docker Image') {
            steps {
                
                script{ 
                // This step should not normally be used in your script. Consult the inline help for details.
                withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                    
                    sh "docker push darr3ng/mission:latest"

                }
                    
                }
            }
        }
        
        stage('Deploy to k8') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'DS-EKS', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://257C34DFB4E2A501F29CD6DD10EF8577.gr7.us-east-1.eks.amazonaws.com') {
                    sh "kubectl apply -f ds.yaml -n webapps"
                    sleep 60
            }
            }
        }
        stage('Verify Deployment') {
            steps {
               withKubeConfig(caCertificate: '', clusterName: 'DS-EKS', contextName: '', credentialsId: 'k8-token', namespace: 'webapps', restrictKubeConfigAccess: false, serverUrl: 'https://257C34DFB4E2A501F29CD6DD10EF8577.gr7.us-east-1.eks.amazonaws.com') {
                    sh "kubectl get  pods -n webapps"
                    sh "kubectl get  svc -n webapps"
                    
            }
            }
        }

    }
}
