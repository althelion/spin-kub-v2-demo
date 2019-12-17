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