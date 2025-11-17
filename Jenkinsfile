pipeline{
    agent {
      node {
	label 'slave1'
    }
}

    tools {
         maven 'MAVEN_HOME'
         jdk 'JAVA_HOME'
         git 'GIT_HOME'
    }

    stages{
        stage('pre-build step') {
            steps {
		sh '''
                echo "Pre Build Step for Webhook Trigges the pipeline on push event"
		'''
	    }
	}
        stage('Git Checkout'){
            steps{
                checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[credentialsId: 'github access', url: 'https://github.com/sanketh2611/Project-2D.git']]])
            }
        }
        stage('Build') {
           steps {
        		sh "mvn clean install -DskipTests"
       			 sh "rm -rf target/webapp.war"
       			 sh "mv target/*.war target/webapp.war"
    			}
		}
        stage ('Static Code Analysis') {
             environment {
             scannerHome = tool 'SONAR_SCANNER'
             }
             steps {
                echo 'Running Static Code Analysis'
                 withSonarQubeEnv('SONAR_HOME') {
                 sh '${scannerHome}/bin/sonar-scanner'
                 }
            }
        }
	stage('Jfrog Artifact Upload') {
    steps {
        rtUpload (
            serverId: 'artifactory',
            spec: '''{
                "files": [
                    {
                        "pattern": "target/*.war",
                        "target": "local-backend-repo/simple-maven-project/1.0-SNAPSHOT/",
                        "props": "type=war;env=prod"
                    }
                ]
            }'''
        )
    }
      }
        stage ('Tomcat Deployment') {
           steps {
             script {
                 deploy adapters: [tomcat9(credentialsId: 'tomcat-credentials', path: '', url: 'http://localhost:9090')], contextPath: '/backendapp', onFailure: false, war: 'target/webapp.war' 
                    }
                  }
           }
         stage('post-build step') {
            steps {
	    	sh '''
                echo "Successfull Pipeline for Tomcat Deployment"
	     	'''
	      }
     	}
    
     }
}
