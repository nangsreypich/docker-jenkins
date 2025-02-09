pipeline {
    agent any
    environment {
        
        // MAIN Image Manager
        DOCKER_HUB_REPOSITORY = "nangsreypich"
        DOCKER_HUB_IMAGE = "html_dev_ops_images"
        DOCKER_CREDENTIALS = "docker-hub-credentials"
        CONTAINER_NAME = "html_dev_ops"
        CONTAINER_PORT = "8890"
    
    }
    parameters {
        gitParameter(name: 'TAG', type: 'PT_TAG', defaultValue: '', description: 'Select the Git tag to build.')
        gitParameter(name: 'BRANCH', type: 'PT_BRANCH', defaultValue: '', description: 'Select the Git branch to build.')
          // Parameter for selecting the deployment action
        choice(name: 'ACTION',choices: ['deploy', 'rollback'],description: 'Choose whether to deploy a new version or rollback to a previous version.')
    }
    stages {
        stage('Checkout Code') {
            steps {
                script {
                   try{
                     if (params.TAG) { 
                        echo "Checking out tag: ${params.TAG}"
                        checkout([$class: 'GitSCM',
                            branches: [[name: "refs/tags/${params.TAG}"]],
                            userRemoteConfigs: [[url: 'https://github.com/nangsreypich/docker-jenkins.git']]
                        ])
                      } else {
                          echo "Checking out branch: ${params.BRANCH}"
                          checkout([$class: 'GitSCM',
                              branches: [[name: "${params.BRANCH}"]],
                              userRemoteConfigs: [[url: 'https://github.com/nangsreypich/docker-jenkins.git']]
                          ])
                      }

                      if(params.ACTION == "rollback"){
                            
                            echo "Status: ${params.ACTION} => on Tag: ${params.TAG}"
                      }else {
                          
                          
                        echo "Status: ${params.ACTION} => Building from ${env.CHECKOUT_REF}"
                    }
                    
                   }catch(Exception e) {
                        
                        echo "Error during checkout process : ${e.message }"
                        currentBuild.result = 'FAILURE'
                        throw e
                    }
                }
            }
        }
        stage('Build and Push Docker Image') {
            steps {
                script {
                  try{
                    if(params.ACTION == "rollback"){
                         
                          echo "Status: ${params.ACTION} => on Tag: ${params.TAG}"
                    }else {
                      // Implement your build logic here
                      echo "Status: ${params.ACTION} =>Building from ${env.CHECKOUT_REF}"
                     
                      // Example build command
                      withCredentials([usernamePassword(credentialsId: DOCKER_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                      }
                      sh """
                        docker build -t ${env.DOCKER_HUB_REPOSITORY}/${env.DOCKER_HUB_IMAGE}:${params.TAG} .
                       
                        docker push ${env.DOCKER_HUB_REPOSITORY}/${env.DOCKER_HUB_IMAGE}:${params.TAG}
                      """
                      
                      echo "Status: ${params.ACTION} => Build done of ${env.DOCKER_HUB_REPOSITORY}/${env.DOCKER_HUB_IMAGE}:${params.TAG} and Push to: ${env.DOCKER_HUB_REPOSITORY}/${env.DOCKER_HUB_IMAGE}:${params.TAG} "
                    }
                  }catch(Exception e) {
                      
                      echo "Error during checkout process : ${e.message}"
                      currentBuild.result = 'FAILURE'
                      throw e
                  } 
                }
            }
        }
        stage("Remove Old Contianer"){
            steps{
                script{
                    try{
                      
                        // Example command to remove a container
                        echo "Removing old container"
                        // docker rm -f ${env.CONTAINER_NAME}
                        def commandWrite = """
                            docker ps -q --filter "name=$CONTAINER_NAME" | grep -q . && docker stop $CONTAINER_NAME && docker rm $CONTAINER_NAME || echo "No running container to remove"
                        """
                        def status = sh(script: commandWrite, returnStatus: true)
                        if(!status){
                 
                            echo "Status: ${params.ACTION} => Removed old container ${env.CONTAINER_NAME}"
                        }else {
                            currentBuild.result = 'FAILURE'
                            throw e
                        }
                    }catch(Exception e) {
                      
                      echo "Error during checkout process : ${e.message}"
  
                      currentBuild.result = 'FAILURE'
                      throw e
                  }  
                }
            }
        }
        stage("Deploying"){
            steps{
              script{
                  try{
                   
                    
                    def commandWrite = """
                        docker run -d --name ${env.CONTAINER_NAME} -p ${env.CONTAINER_PORT}:80 ${env.DOCKER_HUB_REPOSITORY}/${env.DOCKER_HUB_IMAGE}:${params.TAG}
                    """
                    def status = sh(script: commandWrite, returnStatus: true)
                    if(!status){
                        
                        echo "Status: ${params.ACTION} => Deployed ${env.DOCKER_HUB_REPOSITORY}/${env.DOCKER_HUB_IMAGE}:${params.TAG} to ${env.CONTAINER_NAME}"
                    }else {
                      currentBuild.result = 'FAILURE'
                      echo "Error during checkout process : ${e.message}"
                      throw e
                    }
                  }catch(Exception e) {
                    //   sendTelegramMessage("Error during checkout process : ${e.message}")
                      echo "Error during checkout process : ${e.message}"
                      currentBuild.result = 'FAILURE'
                      throw e
                  }  
              }
            }
        } 
    }
    post {
         success {
            echo "✅ Build Successful!"
        }
        failure {
            echo "❌ Build Failed!"
        }
        always {
            echo "🔄 This runs no matter what."
        }
    }
}
