pipeline {

        agent any

	stages {

                stage('Checkout Git') {
                        steps {
                        	git branch: 'main', changelog: false, poll: false, url: 'https://github.com/max-az-10/greenpiece.git'
			}
                }
	}
}
