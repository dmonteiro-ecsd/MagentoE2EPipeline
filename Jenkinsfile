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

        sh "rsync -a magento/* /var/lib/jenkins/workspace/Magento/"

        sh "git clone https://github.com/tschifftner/magento2-deployscripts.git"

        stage 'Tool Setup'
        sh "php -v"
        sh "apt-get install postfix"
        sh "service postfix start"
        // Composer deps like deployer
        sh "composer.phar install"

        // Phing
        if (!fileExists('phing-latest.phar')) {
            sh "curl -sS -O https://www.phing.info/get/phing-latest.phar"
        }
        sh "phing -v"
        sh "printenv"

        stage 'Magento Setup'
        // before install script
        sh "./dev/travis/before_install.sh"
        sh "composer.phar install --no-interaction --prefer-dist"
        //after install script
        sh "./dev/travis/before_script.sh"

        stage 'Asset Generation'
        if (GENERATE_ASSETS == 'true') {
            sh "phing deploy:switch-to-production-mode"
            sh "phing deploy:compile"
            sh "phing deploy:static-content"
            sh "bash bin/build_artifacts_compress.sh"

            archiveArtifacts 'config.tar.gz'
            archiveArtifacts 'var_di.tar.gz'
            archiveArtifacts 'var_generation.tar.gz'
            archiveArtifacts 'pub_static.tar.gz'
            archiveArtifacts 'shop.tar.gz'
        }

        stage 'Deployment'
        if (DEPLOY == 'true') {
            sshagent (credentials: [jenkinsSshCredentialId]) {
                sh "./dep deploy --tag=${TAG} ${STAGE}"
            }
        }

    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    }
}