
def setBuildStatus(String message, String state, String context, String sha) {
    step([
            $class: "GitHubCommitStatusSetter",
            reposSource: [$class: "ManuallyEnteredRepositorySource", url: "https://github.com/<yourRepoURL>"],
            contextSource: [$class: "ManuallyEnteredCommitContextSource", context: context],
            errorHandlers: [[$class: "ChangingBuildStatusErrorHandler", result: "UNSTABLE"]],
            commitShaSource: [$class: "ManuallyEnteredShaSource", sha: sha ],
            statusBackrefSource: [$class: "ManuallyEnteredBackrefSource", backref: "${BUILD_URL}flowGraphTable/"],
            statusResultSource: [$class: "ConditionalStatusResultSource", results: [[$class: "AnyBuildResult", message: message, state: state]] ]
        ]);
}

node('ondemand') {
    try {
        timestamps {
            stage('checkout') {
                print 'branch is ' + scm.branch
                checkout(scm)
                sh './clone.py'
                return vars;
            }
            stage('compile') {
                withMaven(
                    maven: '3.5.0',
                    mavenSettingsConfig: 'maven-settings',
                    mavenLocalRepo: '.repository') {
                    sh "mvn -nsu -Paddons,distrib compile -DskipTests"
                }
            }
            stage('test') {
                withMaven(
                    maven: '3.5.0',
                    mavenSettingsConfig: 'maven-settings',
                    mavenLocalRepo: '.repository') {
                    sh "mvn -nsu -Paddons,distrib test"
                }
            }
            stage('package') {
                withMaven(
                    maven: '3.5.0',
                    mavenSettingsConfig: 'maven-settings',
                    mavenLocalRepo: '.repository') {
                    sh "mvn -nsu -Paddons,distrib -DskipTests install"
                }
            }
        }
    } catch(e) {
        currentBuild.result = "FAILURE"
        //step([$class: 'ClaimPublisher'])
        // mail (to: 'ecm-qapriv@lists.nuxeo.com', subject: "${env.JOB_NAME} (${env.BUILD_NUMBER}) - Failure!", body: "Build failed ${env.BUILD_URL}.")
        // slackSend channel: '#mobile-notifs', color: 'danger', message: "${env.JOB_NAME} - #${env.BUILD_NUMBER} Failure (<${env.BUILD_URL}|Open>)"
        setBuildStatus(
            message: "Failed to build on Nuxeo CI",
            state: 'FAILURE',
            context: 'pull request',
            sha: env.GIT_COMMIT
        )
        throw e
    }
}
