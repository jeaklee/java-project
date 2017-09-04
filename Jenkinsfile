pipeline {
	agent none

	options {
		buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
	}


	stages {
		stage('Unit Tests') {
			agent { label 'apache' }
			steps {
				sh 'ant -f test.xml -v'
				junit 'reports/result.xml'
			}
		}


		stage('build') {
			agent { label 'apache' }
			steps {
				sh 'ant -f build.xml -v'
			}
			post {
				success {
					archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
				}
			}
		}
		stage('deploy') {
			agent { label 'apache' }
			steps {
				sh "cp dist/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/"
			}
		}
		stage('Testing in CentOS') {
			agent { label 'CentOS' }
			steps {
				sh "wget http://jmaster.crypby.com/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
				sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 4"
			}
		}

		stage('Testing in Debian Docker container') {
			agent {
				docker 'openjdk:8u141-jre'
			}
			steps {
				sh "wget http://jmaster.crypby.com/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar"
				sh "cat /etc/issue ; uname -a"
				sh "java -jar rectangle_${env.BUILD_NUMBER}.jar 3 5"
			}
		}

		stage('Promote to Green Status') {
			agent { label 'apache' }
			when {
				branch 'development'
			}
			steps {
				sh "cp /var/www/html/rectangles/all/rectangle_${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/"
			}
		}

	}
}
