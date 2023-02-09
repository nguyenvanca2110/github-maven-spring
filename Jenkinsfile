pipeline{

	agent any

	tools {
	    maven 'Maven_3.9'
	    jdk 'JDK17'
	}

	environment {
		ACR_CREDENTIALS=credentials('KAKASHI_AZURE_ACR')
		ACR_GRADLE_DEMO_URL = "gradlecanvacr.azurecr.io"
		ACR_GRADLE_DEMO = "gradlecanvacr"
		APPLICATION_NAME = "maven-spring-app"
		COMMIT_HASH = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
		NAME_SPACE = "maven-namespace"
        //BRANCH = "${env.GIT_BRANCH}"
        //TAG = "${env.BRANCH}.${env.BUILD_NUMBER}".drop(15)
        //DEV_TAG = "${env.BRANCH}.${env.BUILD_NUMBER}".drop(7)
        //#VERSION = "${env.TAG}"
	}

	stages {

        stage('Maven build image') {
        	steps {
        	    sh 'echo $PWD'
        		// sh 'cd ..'
        		// sh 'echo $PWD'
        		sh 'mvn clean'
        		sh 'mvn package'
        	}
        }

        stage('Docker build image') {
            steps {
                sh 'docker build -t $ACR_GRADLE_DEMO_URL/$APPLICATION_NAME:$COMMIT_HASH .'
        	}
        }

		stage('Docker Hub Login') {
			steps {
				sh 'docker login $ACR_GRADLE_DEMO_URL -u $ACR_CREDENTIALS_USR -p $ACR_CREDENTIALS_PSW'
				// sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
			}
		}

		stage('Docker Image Push') {
			steps {
				sh 'docker push $ACR_GRADLE_DEMO_URL/$APPLICATION_NAME:$COMMIT_HASH'
			}
		}

		stage('Maven K8S Deploy') {
		    steps{
		        script {
		            //withKubeConfig([credentialsId: 'AZURE_AKS', serverUrl: '']) {
                        sh ('kubectl apply -f aks_deployment/aks-namespace.yml')
                        sh ('kubectl apply -f aks_deployment/aks-service-deployment.yml')
                        //sh ('kubectl apply -f aks_deployment/aks-application-deployment.yml')
                        sh ('kubectl set image deployment/$APPLICATION_NAME $APPLICATION_NAME=$ACR_GRADLE_DEMO_URL/$APPLICATION_NAME:$COMMIT_HASH --namespace $NAME_SPACE')
                    //}
                }
            }
        }
	}

	post {
		always {
			sh 'docker image prune -f --filter="dangling=true"'
			sh 'docker rmi -f $ACR_GRADLE_DEMO_URL/$APPLICATION_NAME:$COMMIT_HASH'
			sh 'docker images'
		}
	}


}
