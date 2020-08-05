pipeline {
  agent {
    label 'maven'
  }
  // this is to trigger commit and merge to branch	
  environment {
    DEPLOY_NS_SIT = "${env.DEPLOY_NS_SIT}"
    DEPLOY_NS_UAT = "${env.DEPLOY_NS_UAT}"
	  
    // GIT_FALSE_FULL_NAME =  "${env.GIT_BRANCH,fullName=false}"
    MY_ORI_GIT = "${env.GIT_BRANCH}"
    // MY_NEW_GIT = MY_ORI_GIT.substring(7)
    MY_NEW_GIT = 'MYD-42'
    TWISTLOCKPW = "${env.TWISTLOCK_PASS}"	  

  }
  stages {
	  
  		  
      stage('Update Jira#0 with GitBranch') {
     //  when {
     //     not {
     //       branch 'master'
     //     }
     //  }
      steps {
        script {
		  
		 echo "GIT_BRANCH :" +  "${env.GIT_BRANCH}"
		// echo "MY_NEW_GIT :" +  "${MY_NEW_GIT}"
		// echo "GIT_FALSE : ${GIT_BRANCH,fullName=false} "
		
	           response = jiraAddComment site: 'MyJenkins', idOrKey: "${MY_NEW_GIT}", comment: "Build result: Job - ${JOB_NAME} Build Number = ${BUILD_NUMBER} Build URL - ${BUILD_URL}"
        
        }

      }
    }
	  
     stage('Update Jira#1 with GitBranch') {

      steps {
        script {
		 // MY_ORI_GIT = ${GIT_BRANCH}
		// origin/
		 // MY_NEW_GIT = MY_ORI_GIT.substring(7)
		// echo "GIT_BRANCH :" +  "${GIT_BRANCH}"
	        // echo "MY_NEW_GIT :" +  "${MY_NEW_GIT}"
		
		def issue = jiraGetIssue idOrKey: "${MY_NEW_GIT}", site: 'MyJenkins'
                
		if (issue.code.toString() == '200') {
                     response = jiraAddComment site: 'MyJenkins', idOrKey: "${MY_NEW_GIT}", comment: "Build result: Job - ${JOB_NAME} Build Number = ${BUILD_NUMBER} Build URL - ${BUILD_URL}"
                } else {
                       def issueInfo = [fields: [ project: [key: 'MYD'],
                       summary: "Review build ${MY_NEW_GIT} ",
                       description: 'Review changes for build ${MY_NEW_GIT}',
                       issuetype: [name: 'Task']]]
                        response = jiraNewIssue issue: issueInfo, site: 'MyJenkins'
               }

        }

      }
    }
	  
	  
    // stage('Echoing valuesp') {
    //  steps {
    //    script {
    //      openshift.withCluster() {
    //        echo "DEPLOY_NS :[" + "${env.DEPLOY_NS}" + "]"
    //        echo "Selector Project result :[" + openshift.selector('project', "${DEPLOY_NS}").exists() + "]"
    //        echo "Selector ns result :[" + openshift.selector('ns', "${DEPLOY_NS}").exists() + "]"

     //     }
     //   }
     // }
    // }

    stage('Deleting SIT project if any ..') {
      when {
        expression {
          openshift.withCluster() {
            return openshift.selector('project', "${DEPLOY_NS_SIT}").exists()
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            echo "Deleting SIT project"
            openshift.raw("delete project ${DEPLOY_NS_SIT}")
          }
        }
      }
    }

    stage('Deleting UAT project if any ..') {
      when {
        expression {
          openshift.withCluster() {
            return openshift.selector('project', "${DEPLOY_NS_UAT}").exists()
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            echo "Deleting UAT project"
            openshift.raw("delete project ${DEPLOY_NS_UAT}")
          }
        }
      }
    }

	  
    stage('Artifactory configuration') {
      steps {
        rtServer(
        id: "ARTIFACTORY_SERVER", url: "${env.ARTIFACTORY_SERVER_ID}", credentialsId: "jfrog-credentials")

        rtMavenDeployer(
        id: "MAVEN_DEPLOYER", serverId: "ARTIFACTORY_SERVER", releaseRepo: "libs-release-local", snapshotRepo: "libs-snapshot-local")

        rtMavenResolver(
        id: "MAVEN_RESOLVER", serverId: "ARTIFACTORY_SERVER", releaseRepo: "libs-release", snapshotRepo: "libs-snapshot")
      }
    }

    stage('Maven Clean App') {
      steps {
        rtMavenRun(
        tool: 'maven', // Tool name from Jenkins configuration
        pom: 'pom.xml', goals: 'clean', deployerId: "MAVEN_DEPLOYER", resolverId: "MAVEN_RESOLVER")
      }
    }

   stage('Nexus IQ ') {
      steps {
              echo 'Nexus IQ ..'
	      nexusPolicyEvaluation advancedProperties: '', failBuildOnNetworkError: false, iqApplication: selectedApplication('sandbox-application'), iqStage: 'build', jobCredentialsId: ''
       }
   }
	   
	  
	  
    stage('SonarQube Code Analysis') {

      steps {
        script {

          //  withSonarQubeEnv('sonarscanner') {
          // def mvnHome = tool 'maven'
          // sh "'${mvnHome}/bin/mvn' org.sonarsource.scanner.maven:sonar-maven-plugin:3.3.0.603:sonar -Dsonar.host.url=$SONAR_HOST_URL  -Dsonar.projectKey=${JOB_NAME} -Dsonar.projectName=${JOB_NAME} -Dsonar.language=java -Dsonar.sources=. -Dsonar.java.binaries=target -Dsonar.tests=. -Dsonar.test.inclusions=**/*Test*/* -Dsonar.exclusions=target/**/*.class"
          // }
          withMaven(maven: 'maven') {
            sh "mvn sonar:sonar -Dsonar.language=java -Dsonar.java.binaries=target"
          }

        }

      }
    }

    stage('Maven Install App') {
      steps {
        rtMavenRun(
        tool: 'maven', // Tool name from Jenkins configuration
        pom: 'pom.xml', goals: 'install', deployerId: "MAVEN_DEPLOYER", resolverId: "MAVEN_RESOLVER")
      }
    }

    stage('Creating SIT and UAT project') {
      steps {
        script {
          openshift.withCluster() {

            try {
              // openshift.withCredentials('Jenkins01-token') {
              echo "Creating SIT Project.. "
	      openshift.newProject("${DEPLOY_NS_SIT}")
              
              // ...
              // }
            } catch(e) {
              // The exception is a hudson.AbortException with details
              // about the failure.
              echo "Error encountered in SIT : ${e}"
            }

	    try {
              // openshift.withCredentials('Jenkins01-token') {
              echo "Creating UAT Project.. "
	      openshift.newProject("${DEPLOY_NS_UAT}")
              
              // ...
              // }
            } catch(e2) {
              // The exception is a hudson.AbortException with details
              // about the failure.
              echo "Error encountered in UAT: ${e2}"
            }
		  
		  
          }
        }
      }
    }

    stage('Create SIT Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_SIT}") {
              return ! openshift.selector("bc", "mapit").exists();
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_SIT}") {
              openshift.newBuild("--name=mapit", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
            }
          }
        }
      }
    }
	  
    stage('Create UAT Image Builder') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_UAT}") {
              return ! openshift.selector("bc", "mapit").exists();
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_UAT}") {
              openshift.newBuild("--name=mapit", "--image-stream=redhat-openjdk18-openshift:1.1", "--binary")
            }
          }
        }
      }
    }
    stage('Build SIT Image ') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_SIT}") {
              openshift.selector("bc", "mapit").startBuild("--from-file=target/mapit-spring.jar", "--wait")
            }
          }
        }
      }
    }

    stage('Build UAT Image ') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_UAT}") {
              openshift.selector("bc", "mapit").startBuild("--from-file=target/mapit-spring.jar", "--wait")
            }
          }
        }
      }
    }
    stage('Promote to SIT') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_SIT}") {
              openshift.tag("mapit:latest", "mapit:dev")
            }
          }
        }
      }
    }
    stage('Create SIT deployment') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_SIT}") {
              return ! openshift.selector('dc', 'mapit-dev').exists()
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_SIT}") {
              openshift.newApp("mapit:latest", "--name=mapit-dev").narrow('svc').expose()
            }
          }
        }
      }
    }
    stage('Promote to UAT') {
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_UAT}") {
              openshift.tag("mapit:latest", "mapit:stage")
            }
          }
        }
      }
    }
    stage('Create UAT deployment') {
      when {
        expression {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_UAT}") {
              return ! openshift.selector('dc', 'mapit-stage').exists()
            }
          }
        }
      }
      steps {
        script {
          openshift.withCluster() {
            openshift.withProject("${DEPLOY_NS_UAT}") {
              openshift.newApp("mapit:stage", "--name=mapit-stage").narrow('svc').expose()
            }
          }
        }
      }
    }
    stage('Update Jira') {

      steps {
        script {
          response = jiraAddComment site: 'MyJenkins', idOrKey: 'MYD-7',  comment: "Build result: Job - ${JOB_NAME} Build Number = ${BUILD_NUMBER} Build UL - ${BUILD_URL}"
        }

      }
    }
    
      stage ('Perform Unit Test') {
	     steps {
		     script {
			      openshift.withCluster() {
				      println("Waiting for 20 sec for Pod to be ready")
				      sleep(20) 
				      def hostName = openshift.raw('get route -o jsonpath=\'{.items[0].spec.host}\' -n ${DEPLOY_NS_SIT}') 
				       // println("My hostname" + hostName)
				       // println("Actual hostname" + hostName.actions[0].out)
	                              
				      def response = httpRequest url: 'http://' + hostName.actions[0].out, httpMode: 'GET'
                                       // println("Status: "+response.status)
                                       // println("Content: "+response.content)
				      if ( response.status == 200 ) {
				          println("SIT Unit Test: SUCCESS")
				      } else {
				         println("SIT Unit Test: FAILED" + response.status)
				      }
				      
			      }	      
			     
			     
		     }	     
		     
	     }
       }     
    
    stage('Twistlock ') {
      steps {
        sh '''
              curl -k -ssl -u "bob_the_builder:${TWISTLOCKPW}" https://twistlock-1440-2900.dc-ig-lib-ga-1589529604-f72ef11f3ab089a8c677044eb28292cd-0000.au-syd.containers.appdomain.cloud/api/v1/util/twistcli -o twistcli && chmod +x ./twistcli && ./twistcli images scan --ci --containerized --user bob_the_builder --password ${TWISTLOCKPW} --address https://twistlock-1440-2900.dc-ig-lib-ga-1589529604-f72ef11f3ab089a8c677044eb28292cd-0000.au-syd.containers.appdomain.cloud --details  ${DEPLOY_NS_SIT} 
	   '''
      }
    }  
    
  }
   
}	
