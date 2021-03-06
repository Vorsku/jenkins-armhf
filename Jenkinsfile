pipeline {
	environment {
		PREFIX = 'vorsku'
		REPO = 'jenkins-armhf'
		JENKINS_VERSION = '2.249.3'
		VERSION_LATEST = 'latest'
		
		JENKINS_USER = 'jenkins'
		JENKINS_GROUP = 'jenkins'
		JENKINS_UID = '1000'
		JENKINS_GID = '1000'
		JENKINS_HOME = '/var/jenkins_home'
		JENKINS_HTTP_PORT = '8080'
		JENKINS_AGENT_PORT = '50000'
		TINI_VERSION = 'v0.16.1'
	}
	agent any
	stages {
		stage('Clone source') {
            steps {
                git 'https://github.com/vorsku/jenkins-armhf.git'                
            }
		}	
		stage('Preparation') {
			steps {			    
				sh '''
                mkdir -p docker
				cd docker
                curl -sSLO https://raw.githubusercontent.com/jenkinsci/docker/master/init.groovy
                curl -sSLO https://raw.githubusercontent.com/jenkinsci/docker/master/jenkins.sh
                curl -sSLO https://raw.githubusercontent.com/jenkinsci/docker/master/install-plugins.sh
                curl -sSLO https://raw.githubusercontent.com/jenkinsci/docker/master/plugins.sh
                curl -sSLO https://raw.githubusercontent.com/jenkinsci/docker/master/jenkins-support
                curl -sSLO https://raw.githubusercontent.com/jenkinsci/docker/master/tini_pub.gpg
                curl -sSLO https://raw.githubusercontent.com/jenkinsci/docker/master/tini-shim.sh
				curl -fsSL https://github.com/krallin/tini/releases/download/$TINI_VERSION/tini-static-armhf -o tini
				curl -fsSL https://repo.jenkins-ci.org/public/org/jenkins-ci/main/jenkins-war/$JENKINS_VERSION/jenkins-war-$JENKINS_VERSION.war -o jenkins.war
                chmod +rx *
                cd ..
                '''
			}
		}
		stage ('Build'){
			steps {		
				sh '''
				docker build --rm \
				--file ./Dockerfile.armhf --tag $PREFIX/$REPO:$JENKINS_VERSION-$VERSION_LATEST \
				--tag $PREFIX/$REPO:$VERSION_LATEST \
				--build-arg JENKINS_VERSION=$JENKINS_VERSION \
				--build-arg user=$JENKINS_USER \
				--build-arg group=$JENKINS_GROUP \
				--build-arg uid=$JENKINS_UID \
				--build-arg gid=$JENKINS_GID \
				--build-arg JENKINS_HOME=$JENKINS_HOME \
				--build-arg http_port=$JENKINS_HTTP_PORT \
				--build-arg agent_port=$JENKINS_AGENT_PORT \
				--build-arg JENKINS_SHA=$JENKINS_SHA \
				--no-cache . 
				'''	
			}
		}
		stage('Deploy Image') {
			steps{
				script {
					withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
						sh 'docker push $PREFIX/$REPO:$JENKINS_VERSION-$VERSION_LATEST'
						sh 'docker push $PREFIX/$REPO:$VERSION_LATEST'						
            		}
				}
				sh 'docker rmi $PREFIX/$REPO:$VERSION_LATEST'
			}
		}
		
    }
}
