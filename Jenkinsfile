node {
	try {
		def gradle_home = tool name: 'gradle:4.10.3', type: 'gradle'
		env.PATH = "${gradle_home}/bin:${env.PATH}"
		/*
    	stage('Checkout SCM'){
        	git '/home/samu/GitHub/CITrial'
    	}
    	*/
		stage('Build') {
			tool name: 'gradle:4.10.3', type: 'gradle'
			sh 'gradle build'
		}
		stage('SonarQube Analysis') {
			withSonarQubeEnv('sonarqube') {
				tool name: 'gradle:4.10.3', type: 'gradle'
				sh 'gradle --info sonarqube'
			}
		}
		stage("Quality Gate") {
			timeout(time: 20, unit: 'MINUTES') {
				def qg = waitForQualityGate()
				if (qg.status!='OK' & qg.status!='WARN') {
					error "Pipeline aborted due to quality gate failure: ${qg.status}"
				} else if (qg.status=='WARN') {
					currentBuild.result='UNSTABLE'
					echo "The ${currentBuild.result} state is due to quality gate result: ${qg.status}"
				}
			}
		}
	} catch (e) {
		currentBuild.result = "FAILED"
		throw e
	} finally {
		echo '${currentBuild.result}'
		notifyBuild('${currentBuild.result}')
	}
}


def notifyBuild(def buildStatus) {
    buildStatus =  buildStatus ?: 'FAILURE'
    def colorMap = [ 'SUCCESS': 'good', 'UNSTABLE': 'warning', 'FAILURE': 'danger' ]
    def resultMap = [ 'SUCCESS': 'Success', 'UNSTABLE': 'Unstable', 'FAILURE': 'Failure' ]

    def subject = "${env.JOB_NAME} - #${env.BUILD_NUMBER}"
    echo '${buildStatus}'
    def result = resultMap[buildStatus]
    def summary = "${subject} ${result} after ${currentBuild.durationString.replace(' and counting', '')} (<${env.BUILD_URL}|Details>)"
    def colorName = colorMap[buildStatus]

    slackSend (color: colorName, message: summary)
}