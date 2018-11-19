
node('aliyun-windows-01') {
    stage('CheckoutLatestTestCases') {
        git branch: 'dev',
            credentialsId: 'jenkins-gitlab',
            url:'http://git.xiaoyanzhang.com//automation/api-sanitytest-qa-env'
    }
    try {
        stage('QAEnvTest') {

            env.NODE_ENV = "QA"
            env.API_ROOT_URL = "https://qa.projectname.com"
            env.EXCLUDE1 = "notready"
            env.DIFFY_ENV = "https://qa.projectname.com,https://api.projectname.com"
            env.DIFFY = "NO"
            env.SQL_VALIDATION = "NO"
            print "Environment : ${env.NODE_ENV}"
            bat(script: 'dir', returnStatus: true, returnStdout: true)
            result = bat(script: "pybot --variable QA-API_API_ROOT_URL:${env.API_ROOT_URL} --variable DIFFY_ENV:${env.DIFFY_ENV} --variable DIFFY:${env.DIFFY} --variable SQL_VALIDATION:${env.SQL_VALIDATION} --outputdir ./logs --exclude ${env.EXCLUDE1} ./", returnStatus: true, returnStdout: true)
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
        dingTalk accessToken: '19651dc2dcac41a9da3fedef11621a0f80451c9585f05608ec3cf883ef53e901', jenkinsUrl: "${env.BUILD_URL}", imageUrl: 'https://www.iconsdb.com/icons/preview/soylent-red/x-mark-3-xxl.png', message: " ${env.NODE_ENV} Env Regression Test Fail", notifyPeople: ''
        return
    }
}
