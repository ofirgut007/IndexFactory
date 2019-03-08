pipeline {
	agent any
	environment {
		giturl="git@github.com:ofirgut007/IndexFactory.git"
		branch="master"
		mongoLogin="admin"
		mongoPassword="admin"
		S3bucket="KaymeraS3"
	}   
	options {
		buildDiscarder(logRotator(numToKeepStr:'3'))
		disableConcurrentBuilds()
		skipDefaultCheckout(true)
		timestamps()
	}
	tools {
		// node tool name and config
		nodejs 'MyNode'
	}	
	stages {          
		stage('print') {
			steps {
			echo "Building project with node NPM and publish to ${S3bucket}"
			}
		}
		stage('Checkout') {
			steps {
			deleteDir()
			// Get some code from a GitHub repository	
			echo 'Checking out the git repository of our project'
			git branch: "${branch}", credentialsId: 'github', url: "${giturl}"
			echo "checkout complete"
			}
		}               
		stage('Build') {
			steps {
			echo 'Building project with node NPM'
			INSTALL_NPM()
			//https://www.npmjs.com/package/mongodb
			sh 'npm install mongodb --save'
			sh '''sed -i 's/"admin.system.profile"/self.s.db.databaseName+".system.profile"/g' $WORKSPACE/node_modules/mongodb/lib/admin.js'''
			}
		}
		stage('Run') {
			steps {
			sh 'node server'
			}
		}
		stage('Test') {
			steps {
			sh 'npm test'
			}
		}
		stage('Results') {
			steps {
			echo 'results step'
			//junit '**/target/surefire-reports/TEST-*.xml'
			//archive 'target/*.jar'
			}
		}               

		stage('Publish To s3') {
			steps {
			script {
				echo 'Upload Artifact to s3 according to build number'
				sh 'mkdir -p artifacts'
				sh 'echo test1 > artifacts/${JOB_NAME}-000${BUILD_NUMBER}.txt'
				s3Upload acl: 'PublicReadWrite', bucket: 'ofir-s3-bucket', file: 'artifacts/', metadatas: [''], path: 'Target/'
				echo "Delete old files from workspace"
				dir("${WORKSPACE}")
				{
					deleteDir()
				}
			}
			}
		}    
	}
	post {
		always {
			echo "always running"
		}
		success {
			echo "Notify users when the pipeline successed" 
		}
		failure {
			echo "Notify users when the pipeline fails"
		}
	}
}
void INSTALL_NPM(){
	sh 'sudo apt update -y'
	sh 'sudo apt install nodejs -y'
	sh 'sudo apt install npm -y'
	sh 'npm install'
}
