pipeline {
	agent any
	/*
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
					sh 'gradle sonarqube -Dsonar.host.url=http://sonarqube:9000 -Dsonar.verbose=true'
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
