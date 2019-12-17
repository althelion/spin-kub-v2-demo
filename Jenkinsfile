library('apac_mgmt_jenkins_library')
app = [
    // Docker related
    "app_port": "",
    "host_port": "",
    "dockerArgs": "",

    // Automated deployment
    "unique_identifier": [
        "vm": "",
        "sel-jp": "",
        "sel-au": ""
    ]
]
properties([
    parameters([
        choice(
            name: 'PRODUCT',
            choices: ['sel-jp', 'sel-au', 'spinnaker'],
            description: 'Select a product'
        ),
        choice(
            name: 'ENVIRONMENT',
            choices: ['qa', 'cert', 'prod'],
            description: 'Select an environment'
        )
    ])
])
node {
    try {
        stage('Preparation') {
            tools.clean()
            sh 'env'
            sh 'whoami'
        }
        stage('GitHub: Checkout') {
            checkout([$class: 'GitSCM', branches: [[name: env.BRANCH_NAME]], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: scm.userRemoteConfigs])
            sh 'ls -la'
        }
        stage('SonarQube') {
            def scannerHome = tool 'MySonarQubeScanner'
            withSonarQubeEnv('MySonarQube') {
                sh "${scannerHome}/bin/sonar-scanner"
            }

            script {
                Integer waitSeconds = 10
                Integer timeOutMinutes = 1
                Integer maxRetry = (timeOutMinutes * 60) / waitSeconds as Integer
                for (Integer i = 0; i < maxRetry; i++) {
                    try {
                    timeout(time: waitSeconds, unit: 'SECONDS') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        } else {
                            i = maxRetry
                        }
                    }
                    } catch (Throwable e) {
                    if (i == maxRetry - 1) {
                        throw e
                        }
                    }
                }
            }
        }
        (imageName, imageId) = tools.getImageInfo("go")
        stage('Docker: Build & Push') {
            sh """
            gcloud -v
            docker -v
            """
            dockerTools.push(imageName, imageId)
        }
    } catch (err) {
        tools.handleError(err)
    } finally {
        stage('Clean Up') {
            tools.clean()
        }
    }
}