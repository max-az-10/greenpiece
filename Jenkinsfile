pipeline {

        agent any

	environment {
                SONAR_SCANNER_HOME = tool 'SonarQube Scanner'
	}
	
	stages {

                stage('Checkout Git') {
                        steps {
                        	git branch: 'main', changelog: false, poll: false, url: 'https://github.com/max-az-10/greenpiece.git'
			}
                }

		stage('SonarQube Analysis') {
   			withSonarQubeEnv('SonarQube') {
      				script {
					sh "${SONAR_SCANNER_HOME}/bin/sonar-scanner"
    				}	
  			}
		}
		
	}
}
