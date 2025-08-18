@Library('iti-sharedlib')_

properties([
    disableConcurrentBuilds()
])

node {

    def javaHome = tool name: 'java-11', type: 'jdk'
    def mavenHome = tool name: 'mvn-3-8-8', type: 'maven'

    withEnv([
        "JAVA_HOME=${javaHome}",
        "PATH+JAVA=${javaHome}/bin",
        "PATH+MAVEN=${mavenHome}/bin"
    ]) {

        stage("Get code") {
            checkout scmGit(
                branches: [[name: '*/master']],
                extensions: [],
                userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/java.git']]
            )
        }
        
        stage("Build app") {
            sh 'java -version'
            sh 'mvn -version'
            def mavenBuild = new org.iti.mvn()
            mavenBuild.javaBuild("clean package install")
        }
        
        stage("Archive app") {
            archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
        }
        
        stage("Docker build") {
            def docker = new com.iti.docker()
            docker.build("iti-java", "${BUILD_NUMBER}")
        }
        
        stage("Push image and update manifest") {
            // Use the existing docker-username / docker-password credentials
            withCredentials([
                usernamePassword(
                    credentialsId: 'docker-password', // Jenkins ID for Docker username
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                def docker = new com.iti.docker()
                docker.login("${DOCKER_USER}", "${DOCKER_PASS}")
                docker.push("iti-java", "${BUILD_NUMBER}")
                
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [],
                    userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/argoCD.git']]
                )
                
                sh """
                    sed -i 's#image: .*#image: iti-java:${BUILD_NUMBER}#' deployment.yaml
                    git config user.email "jenkins@your-company.com"
                    git config user.name "Jenkins Automation"
                    git add .
                    git commit -m "Updated image tag to iti-java:${BUILD_NUMBER}" || echo "No changes to commit"
                    git push origin main
                """
            }
        }
    }
}
