@Library('iti-sharedlib')_

properties([
    disableConcurrentBuilds()
])

node {

    def javaHome = tool name: 'java-11', type: 'jdk'
    def mavenHome = tool name: 'mvn-3-5-4', type: 'maven'
    def DOCKER_USER = credentials('docker-username')
    def DOCKER_PASS = credentials('docker-password')

    stage("Get code"){
        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Hassan-Eid-Hassan/java.git']])
    }
    stage("build app"){
        env.JAVA_HOME = javaHome
        env.PATH = "${javaHome}/bin:${mavenHome}/bin:${env.PATH}"
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
        
        stage("update manifest and push to Git"){
            // Checkout the repository with the deployment manifests
            checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Mahmoud-Soudi/argoCD.git']])
            
            // Use sed to update the image tag with the BUILD_NUMBER
            sh "sed -i 's#        image: .*#        image: iti-java:${BUILD_NUMBER}#' deployment.yaml"
            
            // Configure Git user and commit the changes
            sh 'git config user.email "jenkins@your-company.com"'
            sh 'git config user.name "Jenkins Automation"'
            sh 'git add .'
            sh 'git commit -m "Updated image tag to java:${BUILD_NUMBER}"'
            sh 'git push origin main'
        }
    }
}
