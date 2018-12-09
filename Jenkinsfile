node {
	try {
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
		//notifyBuild(currentBuild.result)
	}
}


def notifyBuild(def buildStatus) {
    buildStatus =  buildStatus ?: 'FAILURE'
    env.JOB_DISPLAYNAME = Jenkins.instance.getJob("${env.JOB_NAME}").displayName
    def colorMap = [ 'SUCCESS': 'good', 'UNSTABLE': 'warning', 'FAILURE': 'danger' ]
    def resultMap = [ 'SUCCESS': 'Success', 'UNSTABLE': 'Unstable', 'FAILURE': 'Failure' ]

    def subject = "${env.JOB_DISPLAYNAME} - #${env.BUILD_NUMBER}"
    def result = resultMap[buildStatus]
    def summary = "${subject} ${result} after ${currentBuild.durationString.replace(' and counting', '')} (<${env.BUILD_URL}|Details>)"
    def colorName = colorMap[buildStatus]

    slackSend (color: colorName, message: summary)
}