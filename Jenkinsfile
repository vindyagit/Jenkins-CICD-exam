pipeline {

  environment { // Declaration of environment variables

    DOCKER_ID = "vindyakishore" // replace this with your docker-id

    DOCKER_TAG = "v.${BUILD_ID}.0" // we will tag our images with the current build in order to increment the value by 1 with each new build

  }

  agent any // Jenkins will be able to select all available agents

  stages {

    stage('Docker Build') { // docker build image stage

      steps {

          sh 'docker compose -f docker-compose.yml build'


      }

    }


    stage(' Docker run') { // run container from our built image

      steps {

        script {

          sh '''

          docker compose -f docker-compose.yml up -d

          sleep 10

          '''

        }

      }

    }


    stage('Test Acceptance') { // we launch the curl command to validate that the container responds to the request

      steps {

        script {

          sh '''

          curl localhost:8080

          '''

        }

      }

    }


    stage('Docker Push') { //we pass the built image to our docker hub account

      environment {

            DOCKER_PASS = credentials("DOCKER_HUB_PASS") // we retrieve docker password from secret text called docker_hub_pass saved on jenkins

        }

      steps {

        script {

          sh '''

          docker images

          docker login -u $DOCKER_ID -p $DOCKER_PASS

          docker push vindyakishore/jenkins-cicd-pipeline-movie_service:latest

          docker push vindyakishore/jenkins-cicd-pipeline-cast_service:latest

          docker push vindyakishore/nginx:latest
          
          docker push vindyakishore/postgres:12.1-alpine   

          '''

        }

      }

    }


    stage('Deployment in dev'){

      environment

      {

        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins

      }

      steps {

        script {

          sh '''

          rm -Rf .kube

          mkdir .kube

          ls

          cat $KUBECONFIG > .kube/config

          cp fastapiapp/values.yaml values.yml

          cat values.yml

          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

          helm upgrade --install app fastapi --values=values.yml --namespace dev

          '''

        }

      }

    }


	stage('Deployment in qa'){

      environment

      {

        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins

      }

      steps {

        script {

          sh '''

          rm -Rf .kube

          mkdir .kube

          ls

          cat $KUBECONFIG > .kube/config

          cp fastapiapp/values.yaml values.yml

          cat values.yml

          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

          helm upgrade --install app fastapi --values=values.yml --namespace QA

          '''

        }

      }

    }

	

    stage('Deployment in staging') {

      environment {

        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins

      }

      steps {

        script {

          sh '''

          rm -Rf .kube

          mkdir .kube

          ls

          cat $KUBECONFIG > .kube/config

          cp fastapiapp/values.yaml values.yml

          cat values.yml

          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

          helm upgrade --install app fastapi --values=values.yml --namespace staging

          '''

        }

      }

    }


    stage('Deploiement en prod'){

      environment {

        KUBECONFIG = credentials("config") // we retrieve kubeconfig from secret file called config saved on jenkins

      }

      steps {

      // Create an Approval Button with a timeout of 15 minutes.

      // this requires a manual validation in order to deploy on production environment

        timeout(time: 15, unit: "MINUTES") {

            input message: 'Do you want to deploy in production ?', ok: 'Yes'

        }

        script {

          sh '''

          rm -Rf .kube

          mkdir .kube

          ls

          cat $KUBECONFIG > .kube/config

          cp fastapiapp/values.yaml values.yml

          cat values.yml

          sed -i "s+tag.*+tag: ${DOCKER_TAG}+g" values.yml

          helm upgrade --install app fastapi --values=values.yml --namespace prod

          '''

        }

      }

    }

  }

}
