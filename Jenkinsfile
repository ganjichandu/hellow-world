pipeline {
   agent any
   triggers {
      cron('* * * * *')
    }
   environment{
      ECR_REPO = "hello-world"
	  AWS_ACCT = "474231235995"
	  ECR_URL = "${AWS_ACCT}.dkr.ecr.us-west-2.amazonaws.com/"
	  AWS_RG = "us-west-2"
	  BRANCH = "master"
    }
   stages {
	   stage ('ws cleanup') {
		steps {
		 cleanWs()
		 }
	   }
	   
	   stage ('Code Pull') {
		steps {
		  git branch: 'master', url: 'https://github.com/msquare25/hello-world.git'
		  }
	   }
	   
	   stage ('Remove All Old Docker Images') {
		steps {
		    sh 'images=$(docker images -f dangling=true -q); if [[ ${images} ]]; then docker rmi --force ${images}; fi'
		    }
		  }
		
		stage ('Build war') {
		   steps {
		      sh '''
				mvn clean install package
                '''
		   }
		}
		stage ('Build Images') {
		   steps {
			sh '''/usr/local/bin/aws ecr get-login-password --region $AWS_RG | docker login --username AWS --password-stdin ${AWS_ACCT}.dkr.ecr.us-west-2.amazonaws.com/
			     docker build -t hello-world .
			     docker tag hello-world:latest $ECR_URL$ECR_REPO:$BUILD_NUMBER
			     docker push $ECR_URL$ECR_REPO:$BUILD_NUMBER
				 rm -rf /root/.docker/config.json
			   '''
			}
		}
	    stage ('manifest file Pull') {
		  steps {
		    git branch: 'master', url: 'https://github.com/msquare25/Jenkins-k8s-CICD.git'
		  }
	   }
		stage ('change Images version in manifest file') {
		   steps {
		    sh 'grep -i "image:" k8s-deploy.yml'
			sh 'sed -i "s@image: .*@image: $ECR_URL$ECR_REPO:$BUILD_NUMBER@g" k8s-deploy.yml'
			sh 'grep -i "image:" k8s-deploy.yml'
			}
		}
		stage ('K8S deploy') {
		   steps {
			sshPublisher(publishers: [sshPublisherDesc(configName: 'ansible', transfers: [sshTransfer(cleanRemote: true, excludes: '', execCommand: 'ansible-playbook --become-user ubuntu /home/ansibleadmin/k8s/kubernetes-deployment.yml && ansible-playbook --become-user ubuntu /home/ansibleadmin/k8s/kubernetes-service.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//home//ansibleadmin//k8s', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '*.yml')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
			}
		}
	}
}
