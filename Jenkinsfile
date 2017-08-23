node {
    if (env.BRANCH_NAME != "master") return
    checkout scm
    def authorName = sh(returnStdout: true, script: 'git --no-pager show --format="%an" --no-patch')
    try {
        stage('Compile') {
            sh "mvn clean install -Pjanusgraph-release -Dgpg.skip=true -DskipTests=true"
        }
        stage('Build Image') {
            sh "mvn docker:build -Pjanusgraph-docker -pl janusgraph-dist"
        }
        stage('Push Image') {
            def projectVersion = sh(returnStdout: true, script: 'mvn -q -Dexec.executable="echo" -Dexec.args=\'${project.version}\' --non-recursive exec:exec').trim()
            def tag = "$projectVersion.${env.BUILD_NUMBER}"
            sh "docker tag janusgraph/server:latest docker.blackfynn.io/janusgraph/server:$tag"
            sh "docker push docker.blackfynn.io/janusgraph/server:$tag "
        }
    } catch (e) {
        slackSend(color: '#b20000', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) by ${authorName}")
        throw e
    }
    slackSend(color: '#006600', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL}) by ${authorName}")
}