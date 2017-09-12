#!/usr/bin/env groovy
//to kickstart jenkinsjob

library 'Jenkins_Libraries'

import hudson.model.*
import hudson.EnvVars
import groovy.json.JsonSlurperClassic
import groovy.json.JsonBuilder
import groovy.json.JsonOutput
import java.net.URL

//name of local artifactory server
def server = Artifactory.server 'local_artifactory'
def artifactory_repo = 'myTest'
def scmUrl = scm.getUserRemoteConfigs()[0].getUrl()
version = null

properties ([
    //don't keep build in Jenkins
    buildDiscarder( logRotator(artifactDaysToKeepStr: '',
      artifactNumToKeep: '3',
      daysToKeepStr: '',
      numToKeepStr: '30') ),

    [ $class: 'GithubProjectProperty',
    	displayname: '',
	projectUrlStr: "${getGitHubURL(scmUrl)}" ]
    
])

boolean is_master = ("${env.BRANCH_NAME}" == "master")

    //specify node to build on
	node('builder-bob') {

	try {
			stage('Clean workspace') {
				deleteDir()
				sh 'ls -lah'
			}

			stage('Checkout SCM') {
				checkout scm
			}

	String archive_dir = "output"
	// commented out since it's not used right now
	//String output_dir = "helloworld/${archive_dir}/${getVersion()}/files/"

			stage('Archive File') {
				dir(archive_dir) {
					archiveArtifacts "**"

          if ( is_master ) {

          // define version and set Jenkins display name
          String v = getVersion()
          currentBuild.displayName = "#${env.BUILD_NUMBER} - v${v}"

          // defines artifactory upload spec
          String path = "${artifactory_repo}/${v}-${env.BUILD_NUMBER}"

          String upload_spec = """{
            "files": [
              {
                "pattern": "**",

                "target": "$path/",

                "flat": "false"
              }
           ]
          }"""
          server.upload(upload_spec)
          }
				}
			}

	}

	catch (e) {
		throw e
	}

	finally {

	}

	// fetches "Version_File", read, and pulls version number
	String getVersion() {
		if (!version) {
			version = readFile('VERSION_FILE').replaceAll(/[\n\r]/, '')
		}
		return version
	}
}
}
