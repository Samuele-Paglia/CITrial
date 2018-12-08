pipeline {
	agent ('Gradle') {
		docker {
			image 'gradle'
			args '-v /root/.gradle:/root/.gradle'
		}
	}
	stages {
		stage('Build') {
			steps {
				sh 'gradle build'
			}
		}
		stage('SonarQube Analysis') {
			steps {
				withSonarQubeEnv('sonarqube') {
					sh 'gradle --info sonarqube'
				}
			}
		}
		stage("Quality Gate") {
			agent ('Sonarqube') {
		docker {
			image 'sonarqube:latest'
			args '-p 9000:9000 -v sonarqube-data:/opt/sonarqube/data -v sonarqube-extensions:/opt/sonarqube/extensions'
		}
	}
			steps {
				timeout(time: 20, unit: 'MINUTES') {
					waitForQualityGate abortPipeline: true
				}
			}
		}
	}
}
