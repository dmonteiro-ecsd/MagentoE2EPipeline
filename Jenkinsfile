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
          locations: [[credentialsId: '7ca189fa-9171-4478-a33e-46434f661049', 
                       depthOption: 'infinity', 
                       ignoreExternalsOption: true,  
                       remote: "http://51.140.79.215/svn/magento"]], 
          workspaceUpdater: [$class: 'UpdateUpdater']])

        stage 'Tool Setup'
        sh "${phpBin} -v"
        // Composer deps like deployer
        sh "composer.phar install"
        // Phing
        if (!fileExists('phing-latest.phar')) {
            sh "curl -sS -O https://www.phing.info/get/phing-latest.phar -o ${phingBin}"
        }
        sh "${phingCall} -v"
        sh "printenv"

        stage 'Magento Setup'
        dir('shop') {
            sh "${phingCall} jenkins:flush-all"
            sh "${phingCall} jenkins:setup-project"
            sh "${phingCall} jenkins:flush-all"
        }

        stage 'Asset Generation'
        if (GENERATE_ASSETS == 'true') {
            sh "${phingCall} deploy:switch-to-production-mode"
            sh "${phingCall} deploy:compile"
            sh "${phingCall} deploy:static-content"
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