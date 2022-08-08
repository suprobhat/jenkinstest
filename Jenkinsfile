/*
Requrements:
PR Destination branch should be named as development
credential:  
    BitbucketUser
    SonarQube
Parameter:
    baseBranch: the base branch name


Need to create an additional parameter called 'webhookToken' which will contain stash-sonar-webhook-token    
*/

// def KubeConfigCredentialID      = 'DTKubeConfig'
// def AKSKubeConfigCredentialID   = 'AKSKubeConfig'
def environmentFileName         = 'buildconfig/parameters.groovy'
def K8sDeploymentFiles          = "buildconfig/k8s-deployment-file.yaml"
def ApplicationDirName          = 'DTApplication'

pipeline {
    agent {
        kubernetes {
          cloud "ceedtcluster"
          yaml """
apiVersion: v1
kind: Pod
metadata:
labels:
  component: ci
spec:
  containers:
  - name: jnlp
    image: 618187721717.dkr.ecr.us-east-1.amazonaws.com/docker.io/jenkins/inbound-agent:4.10
    tty: true 
    volumeMounts:  
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: docker
    image: 618187721717.dkr.ecr.us-east-1.amazonaws.com/baseline-repository:docker-latest
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true 
    volumeMounts:  
    - mountPath: /var/run/docker.sock
      name: docker-sock
  - name: sonarscanner
    image: 618187721717.dkr.ecr.us-east-1.amazonaws.com/baseline-repository:sonar-scanner-cli 
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true
  - name: node
    image: 618187721717.dkr.ecr.us-east-1.amazonaws.com/baseline-repository:node-14.17.0
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true
  - name: ceebasictools
    image: 618187721717.dkr.ecr.us-east-1.amazonaws.com/baseline-repository:linux-basic-tools-latest
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true  
  - name: curl
    image: 618187721717.dkr.ecr.us-east-1.amazonaws.com/baseline-repository:alpine-curl
    imagePullPolicy: IfNotPresent
    command:
    - cat
    tty: true  
  volumes:
    - name: docker-sock
      hostPath:
        path: /var/run/docker.sock
"""
        }
    }

    options {
    	buildDiscarder(logRotator(numToKeepStr: '15', artifactNumToKeepStr: '15'))
  	}    
          
	triggers {
		GenericTrigger(
			// Pull request is opened, modified, edited, reopened, synchronize
			genericVariables: [
				[key: 'pr_action', value: '$.pull_request.state'],
				[key: 'repository', value: '$.pull_request.head.repo.clone_url'],
				[key: 'pr_branch', value: '$.pull_request.head.ref'],
				[key: 'base_branch', value: '$.pull_request.base.ref']
			],

			regexpFilterText: '$pr_action:$base_branch',
			regexpFilterExpression: "(?:merged|opened|modified|edited|reopened|synchronize):refs/heads/development", 
			token: "soi-ucs-angular", // Need to create "webhookToken" parameter manually during build plan creation.
			causeString: 'Triggered',
			printContributedVariables: true,
			printPostContent: true,
			silentResponse: false
		)
	}

    stages {
        stage('Download Application Codebase') {
            steps { 
                dir ("${WORKSPACE}") {
                    sh "mkdir -p ${ApplicationDirName}" 
                    dir("${ApplicationDirName}") {
                        script {
                            container('docker') {
                                // if ( repository_clone_0_name == "http" ) {
                                //     repository_url = sh (script: "echo ${repository_clone_0_href}", returnStdout: true).trim()
                                // } else {
                                //     repository_url = sh (script: "echo ${repository_clone_1_href}", returnStdout: true).trim()
                                // }
                                repository_url = sh "echo ${env.repository}"
                                checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: "${repository_url}", credentialsId: "BitbucketUser"]], 
                                extensions: [[$class: 'CloneOption', shallow: true, depth: 1], [$class: 'CheckoutOption', timeout: 30 ]], 
                                branches: [[name: "${env.pr_branch}"]]], poll: false
                                // if (pr_action == "pr:merged") {
                                //     checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: "${repository_url}", credentialsId: "BitbucketUser"]], 
                                //         extensions: [[$class: 'CloneOption', shallow: true, depth: 1], [$class: 'CheckoutOption', timeout: 30 ]], 
                                //         branches: [[name: "${env.base_branch}"]]], poll: false
                                // } else {
                                //     checkout scm: [$class: 'GitSCM', userRemoteConfigs: [[url: "${repository_url}", credentialsId: "BitbucketUser"]], 
                                //         extensions: [[$class: 'CloneOption', shallow: true, depth: 1], [$class: 'CheckoutOption', timeout: 30 ]], 
                                //         branches: [[name: "${env.pr_branch}"]]], poll: false
                                // } 
                            }                      
                        }
                        load "./${environmentFileName}"
                    }
                }                                 
            }
        }         

        // stage('Run Unit Tests') {
        //     steps {
        //         dir ("${WORKSPACE}/${ApplicationDirName}") {
        //             script {
        //                 container('node') {
        //                     sh """
        //                     apt-get update
        //                     apt-get install chromium -y
        //                     apt-get install ttf-freefont -y
        //                     export CHROME_BIN='/usr/bin/chromium'
        //                     npm install -g @angular/cli@13.3.7
        //                     npm install --force
        //                     ng test
        //                     """
        //                 }                    
        //             }
        //         }
        //     }
        // }

        // stage('Run SCA') {
        //     steps { 
        //         dir ("${WORKSPACE}/${ApplicationDirName}") {
        //             echo "Run static code analysis"
        //             withSonarQubeEnv('SonarQube') {
        //                 script {
        //                     container('sonarscanner') {
        //                         buildVersion = sh (script: "echo ${env.build_version}", returnStdout: true).trim()
        //                         if ( pr_action == "pr:merged" ) {
        //                             projectKey = sh (script: "echo ${env.ApplicationName} | tr '[:upper:]' '[:lower:]'", returnStdout: true).trim()
        //                         } else {
        //                             projectKey = sh (script: "echo ${env.ApplicationName}-pr | tr '[:upper:]' '[:lower:]'", returnStdout: true).trim()
        //                         }
        //                         sh """
        //                             sed -i 's|projectKey_update_here|${projectKey}|g' buildconfig/sonar-project.properties
        //                             sed -i 's|projectName_update_here|${projectKey}|g' buildconfig/sonar-project.properties
        //                             cat buildconfig/sonar-project.properties
        //                             sonar-scanner -Dproject.settings=buildconfig/sonar-project.properties
        //                         """
        //                     }   
        //                 }
        //             }
        //         }  
        //     }
        // }

        // stage ('Quality Gate') {
        //     steps {
        //         sleep(60)                        
        //         timeout(time: 1, unit: 'MINUTES') {
        //             waitForQualityGate abortPipeline: true 
        //         }               
        //     }
        // }
                                        
        stage('Build Docker Image') {            
        //     steps {
        //         dir ("${WORKSPACE}/${ApplicationDirName}") {  
        //             script {
        //                 container('ceebasictools') {
        //                     ecrLogin = sh (script: "aws ecr get-login --no-include-email --region us-east-1", returnStdout: true).trim()
        //                 }
        //                 container('docker') {
        //                     sh "apk add docker-compose"
        //                     sh "$ecrLogin"
        //                     if ( pr_action == "pr:merged" ) {
        //                         sh "docker build --network=host -t ${env.RepositoryName}:${env.ApplicationName}-${env.build_version} -f buildconfig/Dockerfile . "
        //                     } else {
        //                         sh "docker build --network=host -t ${env.RepositoryName}:${env.ApplicationName}-${env.build_version}-PR -f buildconfig/Dockerfile . "
        //                     } 
        //                     sh "docker build --network=host -t ${env.RepositoryName}:${env.ApplicationName}-${env.build_version}-PR -f buildconfig/Dockerfile . "
        //                 }                       
        //             } 
        //         }
        //     }
        // }  

        // stage('Push Docker Image To Repository') {
        //     when {
        //         expression { pr_action == "pr:merged" }
        //     }              
        //     steps { 
        //         dir ("${WORKSPACE}/${ApplicationDirName}") {
        //             script {
        //                 container('docker') {
        //                     sh "apk add docker-compose"
        //                     sh "docker push ${env.RepositoryName}:${env.ApplicationName}-${env.build_version}"
        //                     try {
        //                         sh "docker rmi ${env.RepositoryName}:${env.ApplicationName}-${env.build_version}"
        //                     } catch (err) {
        //                         echo err.getMessage()
        //                         echo "Image can not delete Image deletion is failed, but we will continue."
        //                     }
        //                 }                       
        //             }                
        //         }  
        //     }
        // }

        // stage('trigger Spinnaker') {
        //     when {
        //         expression { pr_action == "pr:merged" }
        //     }
        //     steps {
        //       script {
        //         container('ceebasictools') {
        //           echo "trigger spinnaker"
        //           sh """
        //               curl https://dt-spinnaker.inadev.net/api/v1/webhooks/webhook/soucsAngularFrontend -X POST -H "content-type: application/json" -d '{"pipeline": "soucsAngularFrontend", "parameters": { "tag": "${env.ApplicationName}-${env.build_version}"}}'
        //           """
        //         }
        //       }
        //     }
        // }
    }  

    post { 
        always {
            script {
               if (currentBuild.result == 'SUCCESS') {
                  currentBuild.result = currentBuild.result ?: 'SUCCESS'
                  // notifyBitbucket()
               }
            }
            cleanWs()
        }
    }    	 
}
