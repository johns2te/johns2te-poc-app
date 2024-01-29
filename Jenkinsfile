library 'cb-days@master'
def mvnBuildYaml = libraryResource 'podtemplates/maven/pod.yml'
environment {
    SONAR_CRED = credentials('thunder-sonar')
}

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
        container('jdk11'){
          sh '/home/jenkins/agent/workspace/BES_bes_poc_master/mvnw clean package -T8'
          sh 'ls -l /home/jenkins/agent/workspace/BES_bes_poc_master/target/'
          stash name: 'petclinic-jar', includes: 'target/spring-petclinic-2.2.0.BUILD-SNAPSHOT.jar'
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
        container('jdk11'){
            withCredentials([string(credentialsId: 'thunder-sonar', variable: 'SONAR_SECRET')]) {
              sh "./mvnw clean verify -T8 sonar:sonar \
              -Dsonar.sourceEncoding=UTF-8
              -Dsonar.language=java
              -Dsonar.projectKey=petclinic-1 \
              -Dsonar.host.url=https://sonarqube.cb-demos.io \
              -Dsonar.login=${SONAR_SECRET} \
              -Dsonar.projectName=petclinic-1 \
              -Dsonar.tests=src/test \
              -Dsonar.sources=src/main \
              -Dsonar.junit.reportsPath=target/surefire-reports \
              -Dsonar.surefire.reportsPath=target/surefire-reports \
              -Dsonar.jacoco.reportPath=target/jacoco.exec \
              -Dsonar.java.binaries=target/classes \
              -Dsonar.java.coveragePlugin=jacoco"
            } 
        }
      }
    }
    
    stage('CheckMarx Results') {
      agent none
        steps {
          echo '''[{
            "TotalIssues": 6,
            "HighIssues": 0,
            "MediumIssues": 1,
            "LowIssues": 5,
            "InfoIssues": 0,
            "SastIssues": 6,
            "KicsIssues": 0,
            "ScaIssues": -1,
            "APISecurity": {
                "api_count": 0,
                "total_risks_count": 0,
                "risks": null
            },
            "RiskStyle": "medium",
            "RiskMsg": "Medium Risk",
            "Status": "Completed",
            "ScanID": "8a072853-9594-47e5-a544-d6b2d4af837c",
            "ScanDate": "",
            "ScanTime": "",
            "CreatedAt": "2023-07-06, 13:04:53",
            "ProjectID": "e34d40c7-cd4b-4cba-9794-0004f66a173e",
            "BaseURI": "https://ast.checkmarx.net/projects/e34d40c7-cd4b-4cba-9794-0004f66a173e/overview",
            "Tags": {},
            "ProjectName": "bws_enterpriseservices",
            "BranchName": "testing-2",
            "ScanInfoMessage": "",
            "EnginesEnabled": [
                "sast"
            ]
          }]''' > test.json 
        } // mock out CheckMarx results to be pulled in to CDRO for quality gate criteria
    }
   
    stage('Publish') {
      agent any
        steps {
          unstash 'petclinic-jar'
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
