@Library('iti-sharedlib')_

properties([
    disableConcurrentBuilds()
])

node {

    def javaHome = tool name: 'java-11', type: 'jdk'
    def mavenHome = tool name: 'mvn-3-5-4', type: 'maven'
    def DOCKER_USER = credentials('docker-username')
    def DOCKER_PASS = credentials('docker-password')
    withEnv([
        "JAVA_HOME=${javaHome}",
        "PATH+JAVA=${javaHome}/bin",
        "PATH+MAVEN=${mavenHome}/bin"
    ]) 

    stage("Get code"){
        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/java.git']])
    }
    stage("build app"){
        sh 'java -version'
        sh 'mvn -version'

        def mavenBuild = new org.iti.mvn()
        mavenBuild.javaBuild("package install")

    }
    stage("archive app"){
        archiveArtifacts artifacts: '**/*.jar', followSymlinks: false
    }
    stage("docker build"){
        def docker = new com.iti.docker()
        docker.build("iti-java", "${BUILD_NUMBER}")
    }
    stage("push java app image"){
        def docker = new com.iti.docker()
        docker.login("${DOCKER_USER}", "${DOCKER_PASS}")
        docker.push("iti-java", "${BUILD_NUMBER}")
    }
    stage("push java app image"){
        sh "mkdir argocd"
        sh "cd argocd"
        checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/argoCD.git']])
        sh "sed -i #        image: .*#        image: java:${BUILD_NUMBER}# deployment.yaml"
    }
}
