pipeline {
	agent any
	// You have to specify gradle as a tool, otherwise it will not be found
	tools {
		gradle "gradle:4.10.3"
	}
	/*
	// If you wanna use gradle in a docker container
	agent ('Gradle') {
		docker {
			image 'gradle'
			args '-v /root/.gradle:/root/.gradle'
		}s
	}*/
	stages {
		stage('Build') {
			steps {
				sh 'gradle build'
			}
		}
		stage('SonarQube Analysis') {
			steps {
				withSonarQubeEnv('sonarqube') {
					sh 'gradle sonarqube'
				}
			}
		}
		stage("Quality Gate") {
			steps {
				timeout(time: 20, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
	}
}