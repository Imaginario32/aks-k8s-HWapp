pipeline {
	
	agent any
		
	environment {
    		ACR_LOGINSERVER = credentials('alessio32.azurecr.io')
    		ACR_ID = credentials('/subscriptions/605c917c-0504-4230-bc15-6f0b4df7e266/resourceGroups/azure-k8stest/providers/Microsoft.ContainerRegistry/registries/alessio32')
		ACR_PASSWORD = credentials('Uy3fjhZe0tOUBhMaSwxaoWl2mmi/2Odc')
		}
	
	stages {
		
		stage ('webfrontend - Checkout') {
			steps {
					checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/Imaginario32/aks-k8s-HWapp/tree/main/webfrontend']]])
			}
		}
		stage ('Build, Lint, & Unit Test') {
			steps{
					//exectute build, linter, and test runner here    
					sh '''
					echo "exectute build, linter, and test runner here"
					'''
			}
	}
		stage ('Docker Build and Push to ACR'){
			steps{
					
					sh '''
					#Azure Container Registry config
					REPO_NAME="webfrontend"
					ACR_LOGINSERVER="alessio32.azurecr.io"
					ACR_ID="/subscriptions/605c917c-0504-4230-bc15-6f0b4df7e266/resourceGroups/azure-k8stest/providers/Microsoft.ContainerRegistry/registries/alessio32"
					ACR_PASSWORD="Uy3fjhZe0tOUBhMaSwxaoWl2mmi/2Odc"
					IMAGE_NAME="$ACR_LOGINSERVER/$REPO_NAME:v${BUILD_NUMBER}"

					#Docker build and push to Azure Container Registry
					cd ./webfrontend
					docker build -t $IMAGE_NAME .
					cd ..
					
					docker login $ACR_LOGINSERVER -u $ACR_ID -p $ACR_PASSWORD
					docker push $IMAGE_NAME
					'''
			}
	}
		stage ('Helm Deploy to K8s'){
			steps{
					sh '''
                    #Docker Repo Config
					REPO_NAME="webfrontend"
					ACR_LOGINSERVER="alessio32.azurecr.io"

                	#HELM config
					NAME="webfrontend"
					HELM_CHART="./helm/webfrontend"
					
					#Kubenetes config (for safety, in order to make sure it runs in the selected K8s context)
					KUBE_CONTEXT="k8stest"
					kubectl config --kubeconfig=/var/lib/jenkins/.kube/config view
					kubectl config set-context $KUBE_CONTEXT
					
					#Helm Deployment
					helm --kube-context $KUBE_CONTEXT upgrade --install --force $NAME $HELM_CHART --set image.repository=$ACR_LOGINSERVER/$REPO_NAME --set image.tag=v${BUILD_NUMBER} 
					
					#If credentials are required for pulling docker image, supply the credentials to AKS by running the following:
					#kubectl create secret -n $NAME docker-registry regcred --docker-server=$ACR_LOGINSERVER --docker-username=$ACR_ID --docker-password=$ACR_PASSWORD --docker-email=oyarhak@lohika.com
					'''
				}
		}	
	}

	post { 
		always { 
			echo 'Build Steps Completed'
		}
	}
}