pipeline {
  agent none
  stages {
    stage('Cloning Code') {
      agent any
      steps {
        echo 'Cloning Code'
        checkout scmGit(branches: [[name: '*/main']], extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'juice-shop']], userRemoteConfigs: [[url: 'https://github.com/hashcopy/juiceshop']])
      }
    }
    stage('SCA-Snyk') {
      agent any
      steps {
        echo 'running SCA Snyk tool'
        dir('juice-shop'){
          snykSecurity organisation: 'hashcopy', 
          projectName: 'demo_juice_shop', 
          severity: 'medium', 
          snykInstallation: 'snyktool', 
          snykTokenId: 'snyk-token',
          failOnIssues: false,
          additionalArguments: ' --json'
        }
      }
    }
    stage('SCA-gitleaks') {
      agent any
      steps {
        echo "Running Gitleaks SCA"
        sh "rm juice-shop/gitleaks-report.json || true"
        sh """docker run --rm -v ${workspace}/juice-shop:/path zricethezav/gitleaks:v8.18.2 detect --source="/path" --verbose -r /path/gitleaks-report.json -f json --exit-code 0"""
        archiveArtifacts artifacts: "juice-shop/gitleaks-report.json", followSymlinks: false, onlyIfSuccessful: true
      }
    }
    stage('SCA-trufflehog') {
      agent any
      steps {
        echo 'running trufflehog SCA'
        sh 'rm juice-shop/trufflehog.json || true'
        sh 'docker run --rm trufflesecurity/trufflehog:3.76.3 --json github --repo https://github.com/juice-shop/juice-shop --trace >trufflehog.json'
        archiveArtifacts artifacts: "trufflehog.json", followSymlinks: false, onlyIfSuccessful: true
      }
    }
    stage ('SAST-sonarqube') {
      agent any
      tools {
        nodejs 'node20'
      } 
      environment{
        SCANNER_HOME=tool 'sonar-scanner'
      }
      steps {
        sh 'rm juice-shop/sonarqube.json || true'
        withSonarQubeEnv('sonar-server'){
            sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=devSecOps \
            -Dsonar.projectKey=devSecOps -Dsonar.sources=juice-shop '''
        } 
      }
    }
    stage('Clean-workspace') {
      agent any
      steps {
        echo 'cleaning workspace'
        cleanWs()
      }
    }
    
  }
}