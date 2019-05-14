pipeline {

    /*
     * Run everything on an existing agent configured with a label 'docker'.
     * This agent will need docker, git and a jdk installed at a minimum.
     */
//    agent {
//        node {
//            label 'docker'
//        }
//    }

    agent any

    // using the Timestamper plugin we can add timestamps to the console log
    options {
        timestamps()
    }

    tools {
        maven 'maven_3.6.1'
    }

    environment {

        // Get the Maven tool.
        // ** NOTE: This 'maven_3.6.1' Maven tool must be configured
        // **       in the global configuration.
        // mvnHome = tool 'maven_3.6.1'
        // Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
        ARTIFACT_ID = readMavenPom().getArtifactId()
        VERSION = readMavenPom().getVersion()
        VERSION_NEW = readMavenPom().getVersion()
    }

    stages  {
        stage('Preparation') { // for display purposes
            steps{
                // Get some code from a GitHub repository
                git 'https://github.com/dhananjay12/ci-cd-spring-project.git'
            }
        }
        stage('Build') {
            when {
                not { branch 'master' }
            }
            steps{
                echo "----Building PR----"
                sh "mvn clean verify"
                sh "docker tag dhananjay12/${ARTIFACT_ID}:latest dhananjay12/${ARTIFACT_ID}:${VERSION}"
            }
        }
        stage('RELEASE') {
            when {
                branch 'master'
            }
            steps{
                echo "----Building master----"
                // --batch-mode doesnt ask release version and take defaults from POM
                sh "mvn clean --batch-mode release:prepare"
                sh "docker tag dhananjay12/${ARTIFACT_ID}:latest dhananjay12/${ARTIFACT_ID}:${readMavenPom().getVersion()}"
                sh "mvn clean --batch-mode release:perform -D maven.test.skip=true -D docker.skip"
            }
        }
        stage('Results') {
            steps{
                junit '**/target/surefire-reports/TEST-*.xml'
            }
        }

        stage('Push image') {
            steps{
                echo "Trying to Push Docker Build to DockerHub"
                /*
                You would need to first register with DockerHub before you can push images to your account
            */
//                docker.withRegistry('https://registry.hub.docker.com', 'docker-hub') {
//                    sh "docker push dhananjay12/${ARTIFACT_ID}:${VERSION}"
//                    sh "docker push dhananjay12/${ARTIFACT_ID}:latest"
//                }

                withCredentials([string(credentialsId: 'docker-pwd', variable: 'dockerHubPwd')]) {
                    sh "docker login -u dhananjay12 -p ${dockerHubPwd}"
                }
                sh "docker push dhananjay12/${ARTIFACT_ID}:${VERSION}"
                sh "docker push dhananjay12/${ARTIFACT_ID}:latest"

            }

        }
    }

    post {
        always {
            echo 'One way or another, I have finished'
            deleteDir() /* clean up our workspace */
        }
        success {
            echo 'I succeeeded!'
//            slackSend channel: '#ops-room',
//                    color: 'good',
//                    message: "The pipeline ${currentBuild.fullDisplayName} completed successfully."
        }
        failure {
            echo "Job Failed"
            // notify users when the Pipeline fails
//            mail to: 'team@example.com',
//                    subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
//                    body: "Something is wrong with ${env.BUILD_URL}"
        }
    }
}