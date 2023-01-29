pipeline {
    agent any
    environment{
        RELEASE_NAME = 'chatapp'
        DOMAIN = 'chatapp'
        RESOURCE_GROUP = 'capston-res'   
        CLUSTER_NAME = 'capston-aks'
        IMAGE_REPO_NAME= 'capstonacr'
        IMAGE_NAME = 'chatapp'
        IMAGE_TAG = 'latest'
        IMAGE_URL = "${IMAGE_REPO_NAME}.azurecr.io/${IMAGE_NAME}:${IMAGE_TAG}"
        MIN_REPLICA = 1
        MAX_REPLICA = 3
    }

    stages {
        stage('Poll SCM') {
            steps {
              git credentialsId: 'git-token', url: 'git@github.com:anoopraj467/Capston-CD.git'
            }
        }
        stage('Configure Kubectl') {
           
            steps {
                script {
                    
                    sh'''
                    az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME 
                    kubectl get nodes
                    '''                    
                }
            }
        }
        stage('Deploying Helm Charts'){
                steps{
                    script{
                        dir('helm/'){
                            sh'''
                            HOST_NAME=$(az aks show --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME --query addonProfiles.httpApplicationRouting.config.HTTPApplicationRoutingZoneName | tr -d '"')
                            echo $HOST_NAME
                            HOST_DOMAIN=$DOMAIN.$HOST_NAME
                            echo $HOST_DOMAIN
                            helm upgrade --install $RELEASE_NAME . --set hostname=$HOST_DOMAIN --set image=$IMAGE_URL --set minReplicas=$MIN_REPLICA --set maxReplicas=$MAX_REPLICA
                            '''
                        }
                    }
                }
            }
        stage('Verify Deployment'){
            steps{
                sh'''
                    echo "Waiting for end point..."
                    sleep 10
                    ENTRY_POINT = $(kubectl get ingress -o yaml | grep 'host')
                    ENTRY_POINT = ${ENTRY_POINT#*: }
                    curl -Is http://$ENTRY_POINT | head -l
                    echo "URL: http://$ENTRY_POINT"
                    '''
            }
        }
        
        stage("Clean up"){
            steps{
                sh 'rm /var/lib/jenkins/.kube/config'
            }
        }
        
    }
}
