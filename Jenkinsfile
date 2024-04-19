// def COLOR_MAP = [
//     'SUCCESS': 'good', 
//     'FAILURE': 'danger',
// ]
pipeline {
    
	agent any

	tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
    }

    environment {
        SNAP_REPO = 'vprofile-snapshot'
        NEXUS_USER = 'admin'
        NEXUS_PASS = 'admin'
        NEXUS_REPOSITORY = "vprofile-release"
        NEXUS_VERSION = "nexus3"
        NEXUS_PROTOCOL = "http"
        NEXUSIP = "172.31.23.57"
        NEXUSPORT = "8081"
        CENTRAL_REPO = "vpro-maven-central"
	    NEXUS_REPOGRP_ID    = "vpro-maven-group"
        NEXUS_CREDENTIAL_ID = "nexuslogin"
        ARTVERSION = "${env.BUILD_ID}"
        SONARSERVER = 'sonarserver'
        SONARSCANNER = 'sonarscanner'
        // nexuspass: credentials('nexuspass')
    }
	
    stages{
        
        stage('BUILD'){
            steps {
                sh 'mvn -s settings.xml -DskipTests install'
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
                sh 'mvn -s settings.xml test'
            }
        }

	stage('INTEGRATION TEST'){
            steps {
                sh 'mvn -s settings.xml verify -DskipUnitTests'
            }
        }
		
        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn -s settings.xml checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }

        stage('Sonar Analysis') {
            environment {
                scannerHome = tool "${SONARSCANNER}"
            }
            steps {
               withSonarQubeEnv("${SONARSERVER}") {
                   sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
              }
            }
        }
    // stage('quality gate'){
    //     steps {
    //         timeout(time: 1, unit: 'HOURS') {
    //             waitForQualityGate abortPipeline: true
    //         }
    //     }
    // }
  
        stage("UploadArtifact"){
            steps{
                nexusArtifactUploader(
                  nexusVersion: 'nexus3',
                  protocol: 'http',
                  nexusUrl: "${NEXUSIP}:${NEXUSPORT}",
                  groupId: 'QA',
                  version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                  repository: "${NEXUS_REPOSITORY}",
                  credentialsId: "${NEXUS_CREDENTIAL_ID}",
                  artifacts: [
                    [artifactId: 'vproapp',
                     classifier: '',
                     file: 'target/vprofile-v2.war',
                     type: 'war']
                  ]
                )
            }
        }
         stage('Ansible Deploy to staging'){
            steps {
                ansiblePlaybook([
                inventory   : 'ansible/stage.inventory',
                playbook    : 'ansible/site.yml',
                installation: 'ansible',
                colorized   : true,
			    credentialsId: 'app01',
			    disableHostKeyChecking: true,
                extraVars   : [
                   	USER: "admin",
                    PASS: "admin",
			        nexusip: "172.31.23.57",
			        reponame: "vprofile-release",
			        groupid: "QA",
			        time: "${env.BUILD_TIMESTAMP}",
			        build: "${env.BUILD_ID}",
                    artifactid: "vproapp",
			        vprofile_version: "vproapp-${env.BUILD_ID}-${env.BUILD_TIMESTAMP}.war"
                ]
             ])
            }
        }

    }

//   post {
//         always {
//             echo 'Slack Notifications.'
//             slackSend channel: '#jenkins',
//                 color: COLOR_MAP[currentBuild.currentResult],
//                 message: "*${currentBuild.currentResult}:* Job ${env.JOB_NAME} build ${env.BUILD_NUMBER} \n More info at: ${env.BUILD_URL}"
//         }
//     }
}
