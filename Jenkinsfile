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
            docker.build("mahmoudsoudi/iti-java", "${BUILD_NUMBER}")
        }
        
        stage("Push image and update manifest") {
            withCredentials([
                usernamePassword(
                    credentialsId: 'docker-username', // Jenkins ID for your Docker Hub creds
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )
            ]) {
                def docker = new com.iti.docker()
                docker.login("${DOCKER_USER}", "${DOCKER_PASS}")
                docker.push("mahmoudsoudi/iti-java", "${BUILD_NUMBER}")
                
                // ✅ clean checkout of argoCD repo
                checkout scmGit(
                    branches: [[name: '*/main']],
                    extensions: [[$class: 'WipeWorkspace']], // clear old files
                    userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/argoCD.git']]
                )
                
                sh """
                    sed -i 's#image: .*#image: mahmoudsoudi/iti-java:${BUILD_NUMBER}#' deployment.yaml
                    git config user.email "jenkins@your-company.com"
                    git config user.name "Jenkins Automation"
                    git add deployment.yaml
                    git commit -m "Updated image tag to mahmoudsoudi/iti-java:${BUILD_NUMBER}" || echo "No changes to commit"
                    git push origin main
                """
            }
        }
    }
}
