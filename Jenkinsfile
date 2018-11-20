
node() {
    stage('CheckoutLatestTestCases') {
         checkout(
        [
            $class: 'GitSCM',
            extensions: [               
                [$class: 'CleanCheckout'],              
            ],
            branches: [
                [name: '']
            ], 
            userRemoteConfigs: 
            [[
                credentialsId: 'jingghster', 
                url: 'https://github.com/Jinggit/Small.git',
                refspec: ('+refs/pull-requests/*/from:refs/remotes/origin/pr/*/from'), 
                branch: ('origin/pr/${pullRequestId}/from')
            ]]
        ])

    }
    try {
        stage('QAEnvTest') {

            env.NODE_ENV = "QA"
            env.API_ROOT_URL = "https://api.speedx.com"
            env.EXCLUDE1 = "notready"
            print "Environment : ${env.NODE_ENV}"
            sh(script: 'ls', returnStatus: true, returnStdout: true)
            result = sh(script: "pybot --variable API_API_ROOT_URL:${env.API_ROOT_URL} --outputdir ./logs --exclude ${env.EXCLUDE1} ./Speedx", returnStatus: true, returnStdout: true)
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
