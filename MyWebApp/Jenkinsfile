pipeline {
    
    agent any
    
    tools {
        maven 'Maven3'
    }
    
    stages {
        
        stage ('Checkout') {
            steps {
            checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'd52cddff-7577-4a9f-a112-510917ba2aec', url: 'https://github.com/NkeirukaG/MyApplicationRepo']]])
            }
        }
        
        stage ('Build') {
            steps {
            sh 'mvn clean install -f MyWebApp/pom.xml'
            }
        }
        
        stage ('Code Quality') {
            steps {
            withSonarQubeEnv('SonarQube') {
            sh 'mvn -f MyWebApp/pom.xml sonar:sonar'
                }
            }
        }
        
        stage ('JaCoco') {
            steps {
            jacoco()
            }
        }
        
        stage ('Nexus Upload') {
            steps {
            nexusArtifactUploader artifacts: [[artifactId: 'MyWebApp', classifier: '', file: 'MyWebApp/target/MyWebApp.war', type: 'war']], credentialsId: '40a94482-e052-435d-b822-71da10b66834', groupId: 'com.dept.app', nexusUrl: 'ec2-3-21-33-215.us-east-2.compute.amazonaws.com:8081', nexusVersion: 'nexus3', protocol: 'http', repository: 'maven-snapshots', version: '1.0-SNAPSHOT'
            }
        }
        
        stage ('DEV Deploy') {
            steps {
            deploy adapters: [tomcat9(credentialsId: 'c39666f7-0920-4cab-a8aa-94825651c355', path: '', url: 'http://ec2-3-144-182-177.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ('Slack Notification') {
            steps {
            slackSend channel: 'jenkins-into-slack', message: 'deploy into QA Env success! Please  start testing'
            }
        }
        
        stage ('DEV Approve') {
            steps {
                echo "Taking approval from DEV Manager for QA Deployment"
                timeout(time: 7, unit: 'DAYS') {
                input message: 'Do you want to deploy into QA?', submitter: 'admin'
                }    
            }
        }
        
        stage ('QA Deploy') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'c39666f7-0920-4cab-a8aa-94825651c355', path: '', url: 'http://ec2-3-144-182-177.us-east-2.compute.amazonaws.com:8080')], contextPath: null, war: '**/*.war'
            }
        }
        
        stage ('QA Approval') {
            steps {
                echo "Taking approval from QA manager"
                timeout(time: 7, unit: 'DAYS') {
                input message: 'Do you want to proceed to PROD?', submitter: 'admin,manager_userid'
                }
            }
        }
        
        stage ('Slack Notify QA') {
            steps {
            slackSend channel: 'jenkins-into-slack', message: 'deploy into PROD successful!'
            }  
        }
    }
}
