Jenkins Pipeline Template


```
node('web_regression_slave') {
    stage('Checkout') {
        checkout scm
    }
    try {
        stage('DeployQA') {
            timeout(time: 5, unit: 'DAYS') {
                input message: '同意部署到QA环境?', submitter: ''
            }
            env.NODE_ENV = "QA"
            echo "Deploying to ${env.NODE_ENV}"
        }

    } catch (err) {
        currentBuild.result = 'FAILURE'
        dingTalk accessToken: '11651dc2dcac41a9da3fedef11621a0f80451c9585f05608ec3cf883ef53e90e', jenkinsUrl: "${env.BUILD_URL}", imageUrl: 'https://www.iconsdb.com/icons/preview/soylent-red/x-mark-3-xxl.png', message: "${env.NODE_ENV} Env Build Fail", notifyPeople: ''
        return
    }

    try {
        stage('QAEnvTest') {

            env.NODE_ENV = "QA"
            env.API_ROOT_URL = "https://api.1programmer.com"
            env.EXCLUDE1 = "notready"
            print "Environment : ${env.NODE_ENV}"
            bat(script: 'dir', returnStatus: true, returnStdout: true)
            result = bat(script: "pybot --variable QA-API_API_ROOT_URL:${env.API_ROOT_URL} --outputdir ./logs --exclude ${env.EXCLUDE1} ./", returnStatus: true, returnStdout: true)
            step([$class: 'RobotPublisher',
                disableArchiveOutput: false,
                logFileName: 'log.html',
                otherFiles: '',
                outputFileName: 'output.xml',
                outputPath: './logs',
                passThreshold: 100,
                reportFileName: 'report.html',
                unstableThreshold: 100
            ]);

            if (result != 0) {
                echo "${env.NODE_ENV} ${result} Fail"
                echo "${env.BUILD_URL}"
                error("${result} Testcase Failed")
            }
        }
    } catch (err) {
        currentBuild.result = 'FAILURE'
        dingTalk accessToken: '11651dc2dcac41a9da3fedef11621a0f80451c9585f05608ec3cf883ef53e90e', jenkinsUrl: "${env.BUILD_URL}", imageUrl: 'https://www.iconsdb.com/icons/preview/soylent-red/x-mark-3-xxl.png', message: " ${env.NODE_ENV} Env Regression Test Fail", notifyPeople: ''
        return
    }

    try {
        stage('PreTest') {
            timeout(time: 5, unit: 'DAYS') {
                input message: '同意执行Pre环境的测试?', submitter: ''
            }
            env.NODE_ENV = "Pre"
            env.API_ROOT_URL = "https://pre-api.1programmer.com"
            env.EXCLUDE1 = "notready"
            print "Environment : ${env.NODE_ENV}"
            bat(script: 'dir', returnStatus: true, returnStdout: true)
            bat(script: "pybot --variable QA-API_API_ROOT_URL:${env.API_ROOT_URL} --outputdir ./logs --exclude ${env.EXCLUDE1} ./", returnStatus: true, returnStdout: true)
            step([$class: 'RobotPublisher',
                disableArchiveOutput: false,
                logFileName: 'log.html',
                otherFiles: '',
                outputFileName: 'output.xml',
                outputPath: './logs',
                passThreshold: 100,
                reportFileName: 'report.html',
                unstableThreshold: 0
            ]);
        }

    } catch (err) {
        mail body: "build url: ${env.BUILD_URL}",
            from: 'guanhua@1programmer.com',
            replyTo: 'guanhua@1programmer.com',
            subject: 'build failed',
            to: 'guanhua@1programmer.com'
        throw err
    }

    stage('ProdTest') {
        bat(script: 'echo "prod"', returnStatus: true, returnStdout: true)
    }

}
