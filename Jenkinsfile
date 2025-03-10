pipeline {
    agent any
     triggers {
        pollSCM "* * * * *"
     }
    environment {
        GITHUB = credentials('gitHubCredentials')

    }

    stages {
        stage('Checkout') {
            steps{
                checkout scm
            }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }


        stage('Build Docker Image') {
                            when {
                                branch 'master'
                            }

                            steps {
                                echo '=== Building simple-java-maven-app Docker Image ==='
                                script {
                                    app = docker.build("akshaygirpunje/simple-java-maven-app")
                                }
                            }
                }
                stage('Push Docker Image') {
                            when {
                                branch 'master'
                            }
                            steps {
                                echo '=== Pushing simple-java-maven-app Docker Image ==='
                                script {
                                    GIT_COMMIT_HASH = sh (script: "git log -n 1 --pretty=format:'%H'", returnStdout: true)
                                    SHORT_COMMIT = "${GIT_COMMIT_HASH[0..7]}"
                                    docker.withRegistry('https://registry.hub.docker.com', 'dockerHubCredentials') {
                                        app.push("$SHORT_COMMIT")
                                        app.push("latest")
                                    }
                                }
                            }
                }
                stage('Remove local images') {
                            steps {
                                echo '=== Delete the local docker images ==='
                                sh("docker rmi -f akshaygirpunje/simple-java-maven-app:latest || :")
                                sh("docker rmi -f akshaygirpunje/simple-java-maven-app:$SHORT_COMMIT || :")
                }
            }
    }
}
