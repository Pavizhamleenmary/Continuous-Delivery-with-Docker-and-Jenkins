pipeline {
    agent any
    tools {
        maven 'maven 3.9.6'
        gradle 'gradle' // Specify the Gradle tool version
    }
    stages {
        stage('Checkout and Build') {
            steps {
                script {
                    // Checkout the repository
                    checkout([$class: 'GitSCM', branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Pavizhamleenmary/Continuous-Delivery-with-Docker-and-Jenkins']]])

                    // Determine whether it's a Maven or Gradle project
                    def isMaven = fileExists('pom.xml')
                    def isGradle = fileExists('build.gradle')

                    if (isMaven) {
                        echo 'Building Maven project...'
                        sh 'mvn clean package' // Package the application into a JAR file
                    } else if (isGradle) {
                        echo 'Building Gradle project...'
                        sh 'gradle clean build' // Execute Gradle build
                    } else {
                        error 'Unable to identify build tool. No pom.xml or build.gradle found.'
                    }

                    // Copy the JAR file to S3 bucket
                    def jarFileName = ''
                    if (isGradle) {
                        dir('build/libs') {
                            // Find the JAR file dynamically
                            jarFileName = sh(script: 'ls *.jar', returnStdout: true).trim()
                        }
                    } else {
                        dir('target') {
                            // Find the JAR file dynamically
                            jarFileName = sh(script: 'ls *.jar', returnStdout: true).trim()
                        }
                    }

                    // Copy the JAR file to S3 bucket
                    withCredentials([
                        [
                            $class: 'AmazonWebServicesCredentialsBinding',
                            credentialsId: 'AWSCredentials',
                            accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                            secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                        ]
                    ]) {
                        sh "aws s3 cp ${jarFileName} s3://citbucketlatest/"
                    }
                }
            }
        }
        // Other stages here...
    }
}
