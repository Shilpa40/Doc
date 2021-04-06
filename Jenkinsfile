pipeline {
    agent any   
    stages {
        stage('Fetch')
        {
            steps
            {
                git url : "https://github.com/Shilpa40/MavenappSourceCode.git"
            }
        }
        stage('Build')
        {
            steps
            {
                bat 'mvn clean install'
            }
        }
        stage('Sonar Analysis')
        {
            steps
            {
                withSonarQubeEnv("SonarQube")
                {
                    bat "mvn sonar:sonar"
                }  
            }
        }
        stage('Upload to Artifactory')
        {
            steps
            {
                rtMavenDeployer (
                    id: 'deployer-unique-id',
                    serverId: 'Artifactory',
                    releaseRepo: 'example-repo-local',
                    snapshotRepo: 'example-repo-local'
                )
                rtMavenRun (
                pom: 'pom.xml',
                goals: 'clean install',
                deployerId: 'deployer-unique-id'
                )
                rtPublishBuildInfo (
                    serverId: 'Artifactory'
                        )
            }
        }
        stage('Build Image')
                    {
                        steps
                            {
                                 bat "docker build -t dockima:${BUILD_NUMBER} ."
                            }
                    }
         stage ("Pushing the image to dockerhub"){
            steps{
                script{
                        docker.withRegistry('https://registry.hub.docker.com', 'DockerHub') {
                            bat "docker tag dockima:${BUILD_NUMBER}  shilpabains/dockima:${BUILD_NUMBER}"
                            bat "docker rmi dockima:${BUILD_NUMBER}"
                            bat "docker push shilpabains/dockima:${BUILD_NUMBER}"
                }
            }
        }    
      }
        stage("Cleaning Previous Deployment"){
            steps{
                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                            bat "docker stop assignmentdevcontainer"
                            bat "docker rm -f assignmentdevcontainer"
                        }
            }
        }
        stage ("Docker Deployment")
        {
            steps
            {
                bat "docker run --name assignmentdevcontainer -d -p 9056:8080 shilpabains/dockima:${BUILD_NUMBER}"
            }
          }
    }  
  }
