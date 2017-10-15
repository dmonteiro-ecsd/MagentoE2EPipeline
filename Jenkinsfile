node {
    // ENV variables
    env.PWD = pwd()
    env.GENERATE_ASSETS = true
    env.DOCKERIZE = true
    env.DEPLOY = true
    env.PUSH = true 

    try {
        //clean
        sh "chown -R jenkins:jenkins *"
        stage ('Clean') {
            deleteDir()
        }

        // Update Deployment
        checkout([$class: 'SubversionSCM', 
          additionalCredentials: [], 
          excludedCommitMessages: '', 
          excludedRegions: '', 
          excludedRevprop: '', 
          excludedUsers: '', 
          filterChangelog: false, 
          ignoreDirPropChanges: false, 
          includedRegions: '', 
          locations: [[credentialsId: '174efef8-909d-48a1-b6be-8324c7a720a0', 
                       depthOption: 'infinity', 
                       ignoreExternalsOption: true,  
                       remote: "http://51.140.79.215/svn/magento2/"]], 
          workspaceUpdater: [$class: 'UpdateUpdater']])

        stage 'Tool Setup'
        sh "php -v"

        // Phing
        if (!fileExists('phing-latest.phar')) {
            sh "curl -sS -O https://www.phing.info/get/phing-latest.phar"
        }
        sh "phing -v"
        sh "printenv"

        stage 'Magento Setup'
        sh 'cd magento2 && composer install'

        stage 'Asset Generation'

        if (GENERATE_ASSETS == 'true') {
            dir('magento2'){
                sh 'bin/magento module:enable --all --clear-static-content'
                sh 'bin/magento setup:di:compile'
                sh 'tar -cvf magento2.tar.gz .'
            }
        }

        stage 'Dockerize'
  
        if (DOCKERIZE == 'true') {
            dir('magento2'){
                app = docker.build("dmonteiroecsd/magento_docker")
            }
        }

        stage 'Push Docker image'

        if (PUSH == 'true') {
            dir('magento2'){
                docker.withRegistry('https://registry.hub.docker.com', 'docker-credentials') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                }
            }
        }

        stage 'Deployment kube'

        if (DEPLOY == 'true') {
            sh "sudo su - dmonteiro"
            sh 'kubectl run magento-app --image=dmonteiroecsd/magento_docker:latest'
        }
        
        logstashSend failBuild: false, maxLines: 1000

    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    }
}