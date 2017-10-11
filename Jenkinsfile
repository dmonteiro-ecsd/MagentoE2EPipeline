node {
    // ENV variables
    env.PWD = pwd()
    //env.STAGE = STAGE
    //env.TAG = TAG
    //env.REINSTALL_PROJECT = REINSTALL_PROJECT
    //env.DELETE_VENDOR = DELETE_VENDOR
    env.GENERATE_ASSETS = true
    env.DEPLOY = true

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

        //sh "rsync -a magento2/* /var/lib/jenkins/workspace/Magento"
        //sh "sudo cp -Rdfp magento2/* /var/lib/jenkins/workspace/Magento"
        //sh "sudo rm -rf magento2"

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
            sh 'cd magento2 && bin/magento module:enable --all --clear-static-content'
            sh 'cd magento2 && bin/magento setup:di:compile'
            sh 'cd magento2 && tar -cvf magento2.tar.gz .'
        }

        stage 'Dockerize'
  
        if (DEPLOY == 'true') {
            sh 'cd magento2 && sudo docker build -t magento_docker .'
            sh 'echo $(sudo docker images)'
            sh 'echo finally everything is ok'
        }

    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    }
}