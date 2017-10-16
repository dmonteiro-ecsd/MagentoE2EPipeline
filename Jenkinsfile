node {
    // ENV variables
    env.PWD = pwd()
    env.GENERATE_ASSETS = true
    env.DOCKERIZE = true
    env.DEPLOY = true
    env.PUSH = true 
    env.LOGSUPDATE = true
    env.KUBEURL = ''

    try {
        //clean
        sh "sudo chown -R jenkins:jenkins *"
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

        slackSend "Build ${env.BUILD_NUMBER} - code built with success"

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

        slackSend "Build ${env.BUILD_NUMBER} docker image pushed into dmonteiroecsd/magento_docker"

        stage 'Deployment kube'

        if (DEPLOY == 'true') {
            sh "kubectl run magento-app-${env.BUILD_NUMBER} --image=dmonteiroecsd/magento_docker:latest"
            sleep 300
            sh "kubectl expose deployment magento-app-${env.BUILD_NUMBER} --type=LoadBalancer --name=magento-${env.BUILD_NUMBER} --port=80"
            sleep 300
            sh "KUBEDATA=$(kubectl get services magento-${env.BUILD_NUMBER})"
            env.KUBEURL = sh "echo ${KUBEDATA} | cut -d " " -f 10 "
        }

        slackSend "Build ${env.BUILD_NUMBER} - Kubernetes deployment success"
        slackSend "${env.KUBEURL}"
        
        stage 'Update ELKSTACK'

        if (LOGSUPDATE == 'true'){
            logstashSend failBuild: false, maxLines: 10000
        }

        slackSend "ELKStack updated to the build ${env.BUILD_NUMBER}"

        stage 'Clean docker images created'

        sh "docker rmi -f dmonteiroecsd/magento_docker:latest"
        sh "docker rmi -f registry.hub.docker.com/dmonteiroecsd/magento_docker:latest"

    } catch (err) {
        currentBuild.result = 'FAILURE'
        throw err
    }
}