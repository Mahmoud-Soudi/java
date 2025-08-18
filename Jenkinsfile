@Library('iti-sharedlib')_

properties([
    disableConcurrentBuilds()
])

node {

    def javaHome = tool name: 'java-11', type: 'jdk'
    def mavenHome = tool name: 'mvn-3-8-8', type: 'maven'
    def DOCKER_USER = credentials('docker-username')
    def DOCKER_PASS = credentials('docker-password')

    withEnv([
        "JAVA_HOME=${javaHome}",
        "PATH+JAVA=${javaHome}/bin",
        "PATH+MAVEN=${mavenHome}/bin"
    ]) {

        stage("Get code"){
            checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/java.git']])
        }
        
        stage("build app"){
            sh 'java -version'
            sh 'mvn -version'
            def mavenBuild = new org.iti.mvn()
            mavenBuild.javaBuild("clean package install")
        }
        
        stage("archive app"){
            archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
        }
        
        stage("docker build"){
            def docker = new com.iti.docker()
            docker.build("iti-java", "${BUILD_NUMBER}")
        }
        
        stage("push image and update manifest"){
            def docker = new com.iti.docker()
            docker.login("${DOCKER_USER}", "${DOCKER_PASS}")
            docker.push("iti-java", "${BUILD_NUMBER}")
            
            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/argoCD.git']])
            
            sh """
                sed -i 's#image: .*#image: iti-java:${BUILD_NUMBER}#' deployment.yaml
                git config user.email "jenkins@your-company.com"
                git config user.name "Jenkins Automation"
                git add .
                git commit -m "Updated image tag to java:${BUILD_NUMBER}" || echo "No changes to commit"
                git push origin main
            """
        }
    }
}
