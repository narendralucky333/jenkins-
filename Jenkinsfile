#! /usr/bin/env groovy

pipeline {

  agent {
    label 'maven'
  }

  stages {
    stage('Build') {
      steps {
        echo 'Building..'
        
       sh 'mvn clean package'
      }
    }
    stage('Create Container Image') {
      steps {
        echo 'Create Container Image..'
        
        script {

          openshift.withCluster() { 
  openshift.withProject("demo-jenkins") {
  
    def buildConfigExists = openshift.selector("bc", "lucky").exists() 
    
    if(!buildConfigExists){ 
      openshift.newBuild("--name=lucky", "--image=quay.io/narendraprasadn/narendra:latest", "--binary") 
      openshift.newApp("--name=nari", "narendra", "--binary")
    } 
      
    openshift.selector("bc", "lucky").startBuild("--from-file=target/simple.war", "--follow") } }

        }
      }
    }
    stage('Deploy') {
      steps {
        echo 'Deploying....'
        script {

          openshift.withCluster() { 
  openshift.withProject("demo-jenkins") { 
    def deployment = openshift.selector("dc", "lucky") 
    
    if(!deployment.exists()){ 
      openshift.newApp('lucky', "--as-deployment-config").narrow('svc').expose() 
    } 
    
    timeout(5) { 
      openshift.selector("dc", "lucky").related('pods').untilEach(1) { 
        return (it.object().status.phase == "Running") 
      } 
    } 
  } 
}

        }
      }
    }
  }
}
