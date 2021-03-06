pipeline {
	agent none

	environment {
MAJOR_VERSION = 1
	}



	options {
		buildDiscarder(logRotator(numToKeepStr: '2', artifactNumToKeepStr: '1'))
	}


	stages {
		stage('Say hello') {
			agent any

			steps {
				sayHello 'Dear Dan!'
			}
		}

		stage('Git Info') {
			agent any

			steps {
				echo "My branch name: ${env.BRANCH_NAME}"

				script {
					def myLib = new linuxacademy.git.gitStuff();

					echo "My commit: ${myLib.gitCommit("${env.WORKSPACE}/.git")}"
					echo "boo"
				}
			}
		}


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
				sh "mkdir -p /var/www/html/rectangles/all/${env.BRANCH_NAME}"
				sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
			}
		}
		stage('Testing in CentOS') {
			agent { label 'CentOS' }
			steps {
				sh "wget http://jmaster.crypby.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
				sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
			}
		}

		stage('Testing in Debian Docker container') {
			agent {
				docker 'openjdk:8u141-jre'
			}
			steps {
				sh "wget http://jmaster.crypby.com/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
				sh "cat /etc/issue ; uname -a"
				sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 5"
			}
		}

		stage('Promote to Green Status') {
			agent { label 'apache' }
			when {
				branch 'master'
			}
			steps {
				sh "cp /var/www/html/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/"
			}
		}

		stage('Promote development branch to master') {
		agent { label 'apache' }

		when {
			branch 'development'
		}
		steps {
			echo "Stashing local changes"
			sh 'git stash'
			echo 'Checking out development branch'
			sh 'git checkout development'
			echo 'Checking out master branch'
			sh 'git pull origin'
			sh 'git checkout master'
			echo 'Merge dev into master'
			sh 'git merge development'
			echo 'Pushing to Origin Master'
			sh 'git push origin master'
			echo 'Tagging a release'
			sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
			sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
		}
	}
	}

	post {
		failure {
			emailext (
				subject: "${env.JOB_NAME} [${env.BUILD_NUMBER}] Failed!",
				body: """<p>'${env.JOB_NAME} [${env.BUILD_NUMBER}]' Failed!":</p>,
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
				to: "dan.vol4enkov@gmail.com"
				)
		}
	}



}
