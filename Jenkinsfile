pipeline {
    agent any
	
	environment {
		PROJECT_ID = 'intricate-facet-349908'
    CLUSTER_NAME = 'demo-cluster'
    LOCATION = 'us-central1-c'
    CREDENTIALS_ID = 'GKEproject'
    EMAIL_TO = 'swapna.r.nimmala@gmail.com'
	}
	
    stages {	    
	    
	    stage('Test') {
		    steps {
			    echo 'Test'
			   // sh 'mvn test'
		    }
	    }
	    
	    stage('Sonar Qube Analysis') {
		    steps {
			    echo 'Sonar'
		    }
	    }
	    
	    stage('Build Docker Image') {
		    steps {
			     script {
				     myimage = docker.build("swapnarnimmala/apdemo:${env.BUILD_ID}")
				     
				     //sh 'docker build -t swapnarnimmala/apdemo:latest .'
			     }
		    }
	    }
	    
	    stage("Push Docker Image") {
		    steps {
				 //sh 'echo $DOCKERCREDS_PSW | docker login -u $DOCKERCREDS_USR --password-stdin'
				 //sh 'docker push raghukom/devops:latest'
			     script {
				     echo "Push Docker Image"
				     withCredentials([usernamePassword(credentialsId: 'Docker', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                                       sh 'docker login -u $USERNAME -p $PASSWORD'

                            }
				    myimage.push("${env.BUILD_ID}")			    
			     }
		    }
	    }
	    
	    stage('Deploy to K8s Dev') {
		    steps{
			    echo "Deployment started ..."

			     sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
			     sh "sed -i 's/amerisourcebergenapp/amerisourcebergenapp-dev/g' deployment.yaml"

			    // sh 'cat deployment.yaml'
				  echo "Start deployment of deployment.yaml"
				  step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
			    echo "Deployment Finished ..."
		    }
	    }
	    
	    stage('Deploy to K8s Test') {
          steps{
            echo "Deployment started ..."
            sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
            sh "sed -i 's/amerisourcebergenapp-dev/amerisourcebergenapp-test/g' deployment.yaml"
        
          echo "Start deployment of deployment.yaml"
          step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
            echo "Deployment Finished ..."
          }
	    }

      stage('Deploy to K8s Prod') {
        //  when {
        //     branch 'master'
        // }
          steps{

            script {
              timeout(time: 10, unit: 'MINUTES') {
                input(id: "Deploy Gate", message: "Deploy to Prod?", ok: 'Deploy')
              }
            }
            echo "Deployment started ..."
            sh "sed -i 's/tagversion/${env.BUILD_ID}/g' deployment.yaml"
            sh "sed -i 's/amerisourcebergenapp-test/amerisourcebergenapp/g' deployment.yaml"
        
            echo "Start deployment of deployment.yaml"
            step([$class: 'KubernetesEngineBuilder', projectId: env.PROJECT_ID, clusterName: env.CLUSTER_NAME, location: env.LOCATION, manifestPattern: 'deployment.yaml', credentialsId: env.CREDENTIALS_ID, verifyDeployments: true])
              echo "Deployment Finished ..."
            }
	    }
	    
    }

    post {
        failure {
            emailext body: 'Check console output at $BUILD_URL to view the results. \n\n ${CHANGES} \n\n -------------------------------------------------- \n${BUILD_LOG, maxLines=100, escapeHtml=false}', 
                    to: "${EMAIL_TO}", 
                    subject: 'Build failed in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER'
        }
        always {
            echo 'always'
        }
        success {
            echo 'success'
        }
    }
}
