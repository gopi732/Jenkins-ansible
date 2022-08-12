pipeline {
  agent any
  stages { 
    stage('CleanUp WorkSpace & Git Checkout') {
      steps {
          // Clean before build
          cleanWs()
          // We need to explicitly checkout from SCM here
          checkout scm
      }
    }
    stage ("Initialize") {
      steps {
          echo "PATH = ${M2_HOME}/bin:${PATH}"
          echo "M2_HOME = /opt/maven"
      }
    }
    stage ("Testing & SonarQube analysis") {
      environment {
	       scannerHome = tool 'SonarQube Scanner'
      }
      steps {
         withSonarQubeEnv('Sonar') {
            sh "mvn clean verify install sonar:sonar -Dsonar.projectKey=Maven \
		 -Dsonar.java.coveragePlugin=jacoco \
                 -Dsonar.jacoco.reportPaths=target/jacoco.exec \
    	         -Dsonar.junit.reportsPaths=target/surefire-reports"
         }
      }
      post {
         success {
            archiveArtifacts artifacts: 'target/*.war' 
         }
      }
    }
    stage('Deploy Atrifacts') {
	  steps {
	      rtUpload (
		 serverId: 'admin',
		 spec: '''{
 			"files" :[
			  {
		            "pattern": "target/*.war",
		            "target": "Maven/"
	         	  }
		        ]
		 }'''
	     )
	 }
     }
    stage ('Deploy Application') {
      steps {
        sh 'ansible-playbook main.yml'
      }
    }  
  }
}
