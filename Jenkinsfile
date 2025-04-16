//@Library('slack') _


/////// ******************************* Code for fectching Failed Stage Name ******************************* ///////
import io.jenkins.blueocean.rest.impl.pipeline.PipelineNodeGraphVisitor
import io.jenkins.blueocean.rest.impl.pipeline.FlowNodeWrapper
import org.jenkinsci.plugins.workflow.support.steps.build.RunWrapper
import org.jenkinsci.plugins.workflow.actions.ErrorAction

// Get information about all stages, including the failure cases
// Returns a list of maps: [[id, failedStageName, result, errors]]
@NonCPS
List<Map> getStageResults( RunWrapper build ) {

    // Get all pipeline nodes that represent stages
    def visitor = new PipelineNodeGraphVisitor( build.rawBuild )
    def stages = visitor.pipelineNodes.findAll{ it.type == FlowNodeWrapper.NodeType.STAGE }

    return stages.collect{ stage ->

        // Get all the errors from the stage
        def errorActions = stage.getPipelineActions( ErrorAction )
        def errors = errorActions?.collect{ it.error }.unique()

        return [ 
            id: stage.id, 
            failedStageName: stage.displayName, 
            result: "${stage.status.result}",
            errors: errors
        ]
    }
}

// Get information of all failed stages
@NonCPS
List<Map> getFailedStages( RunWrapper build ) {
    return getStageResults( build ).findAll{ it.result == 'FAILURE' }
}

/////// ******************************* Code for fectching Failed Stage Name ******************************* ///////

pipeline {
  agent any

  // environment {
  //   deploymentName = "devsecops"
  //   containerName = "devsecops-container"
  //   serviceName = "devsecops-svc"
  //   imageName = "siddharth67/numeric-app:${GIT_COMMIT}"
  //   applicationURL="http://devsecops-demo.eastus.cloudapp.azure.com"
  //   applicationURI="/increment/99"
  // }

  environment {
    deploymentName = "devsecops"
    containerName = "devsecops-container"
    serviceName = "devsecops-svc"
    imageName = "nomadis/numeric-app:${GIT_COMMIT}"
    applicationURL="http://192.168.4"
    applicationURI="/increment/99"
    dockerhubcreds = credentials('docker-hub-secret')
  }

  stages {

    

    stage('Print jenkins Environment Variables'){
      steps {

        sh 'printenv'

      }      
    }

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archiveArtifacts  'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and JaCoCo') {
      steps {
        sh "mvn test"
      }
      post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
        }
      }
    }

    stage('Mutation Tests - PIT') {
      steps {
              sh "mvn org.pitest:pitest-maven:mutationCoverage"
            }

      post {
        always {
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
        }
      }
    }

    stage('SonarQube Static Analysis') {
      
      steps {
        //withSonarQubeEnv() {
          sh "mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.token=sqa_8033b618a69753e6c3f11cf89c51fa982a6ddec7 -Dsonar.host.url=http://192.168.1.186:9000"
        //}
      }
      // steps {
      //   def mvn = tool 'Default Maven';
      //   withSonarQubeEnv() {
      //     sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=numeric-application -Dsonar.projectName='numeric-application' -Dsonar.host.url=http://192.168.1.186:9000"
      //   }
      // }        
    }

    // stage('Build & Push Docker Image') {
    //   steps {
    //     withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
    //       sh 'printenv'
    //       sh 'docker build -t nomadis/numeric-app:""$GIT_COMMIT"" .'
    //       sh 'docker push nomadis/numeric-app:""$GIT_COMMIT""'
    //     }
    //   }
    // }

     stage('Build & Push Docker Image') {
      steps {
           // withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          
          //sh 'sudo nerdctl login -u registryuser -p osboxesdotorg --insecure-registry osboxes:5000'
          //sh 'sudo nerdctl tag numeric-app:""$GIT_COMMIT"" osboxes:5000/numeric-app/numeric-app:""$GIT_COMMIT""'
          //sh 'sudo nerdctl push --insecure-registry osboxes:5000/numeric-app/numeric-app:""$GIT_COMMIT""'
          //sh 'sudo nerdctl login -u nomadis -p "J8e6qY74pEH8qWM" hub.docker.com'
          //sh 'sudo nerdctl save nomadis/numeric-app:""$GIT_COMMIT"" -o /home/osboxes/image.tar'
          //}
       
       
          sh 'printenv'
          sh 'sudo nerdctl build -t nomadis/numeric-app:""$GIT_COMMIT"" .'
          sh 'echo "Done building"'
          
          sh 'sudo nerdctl login -u nomadis -p ${dockerhubcreds}' 
          sh 'sudo nerdctl push nomadis/numeric-app:""$GIT_COMMIT""'
        
      }
    }

    stage('K8S deployment - DEV'){
      steps{
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#nomadis/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          //sh "sed -i 's#replace#osboxes:5000/numeric-app/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          //sh "kubectl apply  --insecure-skip-tls-verify=true -f k8s_deployment_service.yaml"
          sh "kubectl apply  -f k8s_deployment_service.yaml"
          //sh 'kubectl run -p 8080:8090 nomadis/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

 //    stage('Mutation Tests - PIT') {
 //      steps {
 //        sh "mvn org.pitest:pitest-maven:mutationCoverage"
 //      }
 //    }

    

 //    stage('SonarQube - SAST') {
 //      steps {
 //        withSonarQubeEnv('SonarQube') {
 //          sh "mvn sonar:sonar \
	// 	              -Dsonar.projectKey=numeric-application \
	// 	              -Dsonar.host.url=http://devsecops-demo.eastus.cloudapp.azure.com:9000"
 //        }
 //        timeout(time: 2, unit: 'MINUTES') {
 //          script {
 //            waitForQualityGate abortPipeline: true
 //          }
 //        }
 //      }
 //    }

	// stage('Vulnerability Scan - Docker') {
 //      steps {
 //        parallel(
 //        	"Dependency Scan": {
 //        		sh "mvn dependency-check:check"
	// 		},
	// 		"Trivy Scan":{
	// 			sh "bash trivy-docker-image-scan.sh"
	// 		},
	// 		"OPA Conftest":{
	// 			sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-docker-security.rego Dockerfile'
	// 		}   	
 //      	)
 //      }
 //    }
    

 //    stage('Docker Build and Push') {
 //      steps {
 //        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
 //          sh 'printenv'
 //          sh 'sudo docker build -t siddharth67/numeric-app:""$GIT_COMMIT"" .'
 //          sh 'docker push siddharth67/numeric-app:""$GIT_COMMIT""'
 //        }
 //      }
 //    }

 //    stage('Vulnerability Scan - Kubernetes') {
 //      steps {
 //        parallel(
 //          "OPA Scan": {
 //            sh 'docker run --rm -v $(pwd):/project openpolicyagent/conftest test --policy opa-k8s-security.rego k8s_deployment_service.yaml'
 //          },
 //          "Kubesec Scan": {
 //            sh "bash kubesec-scan.sh"
 //          },
 //          "Trivy Scan": {
 //            sh "bash trivy-k8s-scan.sh"
 //          }
 //        )
 //      }
 //    }

 //    stage('K8S Deployment - DEV') {
 //      steps {
 //        parallel(
 //          "Deployment": {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "bash k8s-deployment.sh"
 //            }
 //          },
 //          "Rollout Status": {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "bash k8s-deployment-rollout-status.sh"
 //            }
 //          }
 //        )
 //      }
 //    }

 //    stage('Integration Tests - DEV') {
 //      steps {
 //        script {
 //          try {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "bash integration-test.sh"
 //            }
 //          } catch (e) {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "kubectl -n default rollout undo deploy ${deploymentName}"
 //            }
 //            throw e
 //          }
 //        }
 //      }
 //    }

 //   stage('OWASP ZAP - DAST') {
 //      steps {
 //        withKubeConfig([credentialsId: 'kubeconfig']) {
 //          sh 'bash zap.sh'
 //        }
 //      }
 //    }

 //    stage('Prompte to PROD?') {
 //      steps {
 //        timeout(time: 2, unit: 'DAYS') {
 //          input 'Do you want to Approve the Deployment to Production Environment/Namespace?'
 //        }
 //      }
 //    }

 //    stage('K8S CIS Benchmark') {
 //      steps {
 //        script {

 //          parallel(
 //            "Master": {
 //              sh "bash cis-master.sh"
 //            },
 //            "Etcd": {
 //              sh "bash cis-etcd.sh"
 //            },
 //            "Kubelet": {
 //              sh "bash cis-kubelet.sh"
 //            }
 //          )

 //        }
 //      }
 //    }

 //    stage('K8S Deployment - PROD') {
 //      steps {
 //        parallel(
 //          "Deployment": {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "sed -i 's#replace#${imageName}#g' k8s_PROD-deployment_service.yaml"
 //              sh "kubectl -n prod apply -f k8s_PROD-deployment_service.yaml"
 //            }
 //          },
 //          "Rollout Status": {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "bash k8s-PROD-deployment-rollout-status.sh"
 //            }
 //          }
 //        )
 //      }
 //    }

 //    stage('Integration Tests - PROD') {
 //      steps {
 //        script {
 //          try {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "bash integration-test-PROD.sh"
 //            }
 //          } catch (e) {
 //            withKubeConfig([credentialsId: 'kubeconfig']) {
 //              sh "kubectl -n prod rollout undo deploy ${deploymentName}"
 //            }
 //            throw e
 //          }
 //        }
 //      }
 //    }   
   
      stage('Testing Slack - 1') {
      steps {
          sh 'exit 0'
      }
    }

   stage('Testing Slack - Error Stage') {
      steps {
          sh 'exit 0'
      }
    }

  }

  //post { 
     //    always { 
     //      junit 'target/surefire-reports/*.xml'
     //      jacoco execPattern: 'target/jacoco.exec'
     //      pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
     //      dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
     //      publishHTML([allowMissing: false, alwaysLinkToLastBuild: true, keepAll: true, reportDir: 'owasp-zap-report', reportFiles: 'zap_report.html', reportName: 'OWASP ZAP HTML Report', reportTitles: 'OWASP ZAP HTML Report'])
        
 		  // //Use sendNotifications.groovy from shared library and provide current build result as parameter 
     //      //sendNotification currentBuild.result
     //    }
     
        //success {
        	//script {
		        /* Use slackNotifier.groovy from shared library and provide current build result as parameter */  
		        //env.failedStage = "none"
		       // env.emoji = ":white_check_mark: :tada: :thumbsup_all:" 
		      //  sendNotification currentBuild.result
		     // }
       // }

	   // failure {
	    //	script {
			  //Fetch information about  failed stage
		    //  def failedStages = getFailedStages( currentBuild )
	        //  env.failedStage = failedStages.failedStageName
	        //  env.emoji = ":x: :red_circle: :sos:"
		     // sendNotification currentBuild.result
		   // }	
        
	  //  }
      
  //  }

}