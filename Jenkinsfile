node {
    checkout scm
    if (env.BRANCH_NAME != "master") return
    try {
        stage('Build Image') {
            sh "mvn clean install -Pjanusgraph-release -Dgpg.skip=true -DskipTests=true && mvn docker:build -Pjanusgraph-docker -pl janusgraph-dist"
        }
        stage('Push Image') {
            def projectVersion = sh(returnStdout: true, script: 'mvn -q -Dexec.executable="echo" -Dexec.args=\'${project.version}\' --non-recursive exec:exec')
            def jgImage = image("janusgraph/server:latest")
            jgImage.push("docker.blackfynn.io/blackfynn/janusgraph:$projectVersion.${env.BUILD_NUMBER}")
        }
    } catch (e) {
        slackSend(color: '#b20000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) by ${authorName}")
        throw e
    }
    slackSend(color: '#006600', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) by ${authorName}")
}