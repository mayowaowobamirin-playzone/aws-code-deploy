#!/usr/bin/env groovy
@Library('pipeline-util') _
def  mavenInfo
def deploymentUnitName
def version
def config
def TEST_URL

pipeline {
	agent any
	parameters {
		gitParameter(description:'Branch name to deploy ',branchFilter: 'origin/(.*)', defaultValue: 'master', name: 'BRANCH', type: 'PT_BRANCH')
		string(name: 'MaxServers', defaultValue: '3', description: 'Maximum number of instances')
		string(name: 'MinServers', defaultValue: '2', description: 'Minimum number of instances')
		choice(name: 'TemplateName', choices: [
			'https://s3-us-west-2.amazonaws.com/amar-deep-singh/boot_template.json',
			'https://s3-us-west-2.amazonaws.com/amar-deep-singh/app_launcher.json',
			'https://s3.us-east-2.amazonaws.com/amardeep-singh/app_launcher.json'
		], description: 'Deployment Environment')
	}
	stages {
		stage('Build') {
			steps {
			script{
	mavenInfo = readMavenPom file:''
	deploymentUnitName = mavenInfo.artifactId
	version = mavenInfo.version
	config = readProperties file:'pipeline/config.properties'
	echo "Deployment unit name = ${deploymentUnitName}"
			}

				buildSource()
			}
		}
		
		stage('Analysis') {
			steps {
				performAnalysis()
							}
		}//end of Analysis Stage


		stage('Packaging') {
			steps {
				uploadArtifacts()
			}
		}//end of Packaging Stage

		stage('Automated Testing') {
			steps {
					deployToAws(deploymentUnitName,"Dev")
					performSanityCheck(deploymentUnitName)
			}//end of step

		}//end of Automated Testing stage
		
		stage('Functional Testing') {
			steps {
				timeout(time: 2, unit: 'HOURS') {
					input message: 'Click to Done testing if you are done with your manual testing', ok: 'Done Testing'
				}//end of timeout
				echo 'Tearing down Dev environment'
				terminateAws(deploymentUnitName,"Dev")
				echo 'Deploying System to SIT environment'
			}
		}//end of SIT stage

		stage('UAT') {
			steps { uatDeploy(deploymentUnitName) }
		}//end of UAT stage

		stage('PROD') {
			steps { prodDeploy(deploymentUnitName) }
		}//end of SIT stage
	}//end of stages
	post{
		always {
			echo 'Tearing down Dev environment'
			terminateAws(deploymentUnitName,"Dev")
		}//end of always
	}//end of post
}//end of pipeline
