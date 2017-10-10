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
                       remote: "http://51.140.79.215/svn/magento/"]], 
          workspaceUpdater: [$class: 'UpdateUpdater']])

        sh "rsync -a magento/* /var/lib/jenkins/workspace/Magento"

        stage 'Tool Setup'
        sh "php -v"

        // Phing
        if (!fileExists('phing-latest.phar')) {
            sh "curl -sS -O https://www.phing.info/get/phing-latest.phar"
        }
        sh "phing -v"
        sh "printenv"

        stage 'Magento Setup'

        sh 'composer install'

        stage 'Asset Generation'
        if (GENERATE_ASSETS == 'true') {
            sh 'bin/magento module:enable --all --clear-static-content'
            sh 'bin/magento setup:di:compile'
            sh 'tar -czvf magento2.tar.gz /var/lib/jenkins/workspace/Magento --exclude-vcs --exclude=".[^/]*" --exclude="*.log"'
        }

        stage 'Deployment'
        if (DEPLOY == 'true') {
        //    sshagent (credentials: [jenkinsSshCredentialId]) {
        //        sh "./dep deploy --tag=${TAG} ${STAGE}"
        //    }
        sh 'echo finally everything is ok'
        }

    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    }
}