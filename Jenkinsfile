library 'cb-days@master'
def mvnBuildYaml = libraryResource 'podtemplates/maven/pod.yml'
pipeline {
  agent none
  options { 
    buildDiscarder(logRotator(numToKeepStr: '10'))
    skipDefaultCheckout true
    preserveStashes(buildCount: 10)
  }
  stages {
    stage('Maven Install') {
      agent{
        kubernetes {
          label 'maven-app'
          yaml mvnBuildYaml
        }
      }
      steps {
        checkout scm
        container('maven-jdk8'){
          sh 'mvn package'
          sleep 2000
          sh 'ls -l /home/jenkins/agent/workspace/clinic-apps_sho-petclinic_master/target'
          stash name: 'sho-petclinic-jar', includes: 'spring-petclinic-2.2.0-BUILD-SNAPSHOT.jar'
          
          // sh 'curl -v --ssl-reqd -u praumann:jmUD156yzCxXnCiHxH0V3ktumLHPpfYs -T /home/jenkins/agent/workspace/thunder-petclinic_main/target/spring-petclinic-3.1.0.jar ftp://us-east-1.sftpcloud.io:21/'
        }
      }  
    }
    
    stage('SonarQube Analysis') {
       agent{
        kubernetes {
          label 'maven-app'
          yaml mvnBuildYaml
        }
      }
      steps {
        checkout scm
        container('maven-jdk8'){
          //withSonarQubeEnv(credentialsId: 'petclinic-1-sonar', installationName: 'Sonar')
         // { // You can override the credential to be used
           // sh './mvnw clean org.sonarsource.scanner.maven:sonar-maven-plugin:3.7.0.1746:sonar'
          //}
          
          sh "mvn sonar:sonar \
          -Dsonar.projectKey=petclinic-1 \
          -Dsonar.host.url=https://sonarqube.cb-demos.io \
          -Dsonar.login=13094ff5ed08f3626272650bb019588afeae1dcb \
          -Dsonar.exclusions='**/*.java' \
          -Dsonar.projectName=petclinic-1" 
          sleep 2000
        }
      }
    }
    
//node {
//  stage('SCM') {
//    checkout scm
//  }
//    stage('Test') {
//      agent any
//        steps {
//          junit '**/build/test-reports/*.xml'
//        }
//    }    
    
    stage('Publish') {
      agent any
        steps {
          unstash 'sho-petclinic-jar'
          cloudBeesFlowPublishArtifact artifactName: 'com.cloudbees:petclinic', artifactVersion: '$BUILD_ID', configuration: 'CD', filePath: 'target/*.jar', relativeWorkspace: '', repositoryName: 'default'
          archiveArtifacts artifacts: 'target/*.jar', followSymlinks: false
        }
    } 
    stage('Trigger Release') {
      agent any
        steps {
          cloudBeesFlowTriggerRelease configuration: 'CD', parameters: '{"release":{"releaseName":"' + 'petclinic' + '","stages":"[{\\"stageName\\": \\"Pre-Production\\", \\"stageValue\\": true}, {\\"stageName\\": \\"Production\\", \\"stageValue\\": true}, {\\"stageName\\": \\"Quality Assurance\\", \\"stageValue\\": true}, {\\"stageName\\": \\"Release Readiness\\", \\"stageValue\\": true}]","parameters":"[]"}}', projectName: 'praumann Demo', releaseName: 'petclinic', startingStage: 'Release Readiness'}        
        }
    }
} //pipeline conclusion
