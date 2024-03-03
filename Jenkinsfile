#!groovy
Library('dxraSharedLib')

pipeline {
    agent { label 'Linux-AWS' }

    stages {
        //stage("Checkout") {
        //    steps{
        //        echo "Checkout source code"
        //        git branch: "master"
        //    }
        //}

        stage('Build and test') {
            steps {
                script {
                    buildJava {
                        buildTool = 'maven'
                        mavenConfig = [
                            settingsWithFlag: '',
                            pomFile: 'pom.xml',
                            additional: '',
                        ]
                    } //Run Sonarqube analysis
                    //sh mnv sonar:sonar
                }
            }
            post {
                success {
                    stash includes: 'target/**', name: 'build'
                }
            }
        }
        stage('Code Quality - SonarQube') {
            options {
                timeout(time:10, unit: 'MINUTES')
            }
            steps {
                script {
                    codeQuality {
                        mode = "maven"
                        sonaryKey = 'CorePlataformServices::EventHubCPS::EventHubCPS::SNSVC0063678'
                        sonarHost = 'https://sonarqube02.barcapint.com'
                        sonarCreds = 'EventhubR-Sonar-D'
                        mavenPomFile = "pom.xml"
                        sonarMainBranch = "feature/CSEELQOL-769"
                        javaBinaryPath = 'target/classes'
                        jacocoReportPath = '/target/site/surefire-report.html'
                    }
                }
            }
        }
        stage('Define version') {
            steps {
                script {
                    this.env.RELEASE_VERSION = "${this.env.BUILD_TIMESTAMP}${this.env.BRANCH_NAME == 'feature/CSEEQOL-769' ? '' : '-SNAPSHOT'}"
                }
            }
        }
        stage('Package - Docker') {
            agent {
                label 'Docker-agent'
            }
            steps {
                script {
                    deleteDir()
                    checkout scm

                    unstash 'build'

                    boolean publishLatestDockerTag = (this.env.BRANCH_NAME == 'feature/CSEEQOL-769')
                    dockerBuild {
                        imageName = "application"
                        version = this.env.RELEASE_VERSION
                        credentialsId = 'eventhubcicdtest'
                        nameSpace = 'E_bcp'
                        correlationId = 'Eventhub_itsi'
                        applyLatest = publishLatestDockerTag
                        prismaScan = true
                        dockerFilePath = "${this.env.WORKSPACE}/"
                        dockerFileName = 'Dockerfile'
                        cmdLineArgs = "--build-arg APP_JAR=target/application-${this.env.RELEASE_VERSION}.jar"
                    }
                }
            }
        }
        
    }
	
}