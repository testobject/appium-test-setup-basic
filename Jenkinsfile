#!groovy

def runTest() {
    node {
        stage("checkout") {
            checkout scm
        }
        stage("test") {
            def gradleTestCommand
            if (env.TEST_CLASS) {
                gradleTestCommand = "./gradlew -Dtest.single=${env.TEST_CLASS} clean test"
            } else {
                gradleTestCommand = "./gradlew clean test"
            }
            try {
                docker.image("java:8").inside {
                    sh gradleTestCommand
                }
            } finally {
                junit "**/test-results/*.xml"
            }
        }
    }
}

try {
    runTest()
    if (env.SUCCESS_NOTIFICATION_ENABLED) {
        slackSend channel: "#${env.SLACK_CHANNEL}", color: "good", message: "`${env.JOB_BASE_NAME}` passed (<${BUILD_URL}|open>)", teamDomain: "${env.SLACK_SUBDOMAIN}", token: "${env.SLACK_TOKEN}"
    }
} catch (err) {
    if (env.APPIUM_URL.contains("testobject.com") || env.FAILURE_NOTIFICATION_ENABLED) {
        slackSend channel: "#${env.SLACK_CHANNEL}", color: "bad", message: "`${env.JOB_BASE_NAME}` failed: $err (<${BUILD_URL}|open>)", teamDomain: "${env.SLACK_SUBDOMAIN}", token: "${env.SLACK_TOKEN}"
    }
    throw err
}
