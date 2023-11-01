def nexusDestination            = "10.21.3.24:8083"
def nexusDestinationCloud       = "docker.share.cloud.k2view.com"
def repo_link                   = "https://github.com/k2view-LTD/k2-agent.git"
def repo_branch                 = "master"
def scan_link                   = ""

pipeline {
    agent {label 'master'}
    parameters {
        string(name: 'imageName', description: 'Image name to be set', defaultValue: 'k2view/k2v-agent')
        string(name: 'version', description: 'Image version to be set', defaultValue: '1.0')
        // Nexus
        booleanParam(name: 'upload_to_nexus', description: 'select it you want to upload the image to RND Nexus.', defaultValue: true)
        booleanParam(name: 'upload_to_nexus_as_latest', description: 'select it you want to upload the image to RND Nexus with latest tag.', defaultValue: true)
        // Cloud Nexus
        booleanParam(name: 'upload_to_cloud_nexus', description: 'select it you want to upload the image to cloud shared Nexus.', defaultValue: true)
        booleanParam(name: 'upload_to_cloud_nexus_as_latest', description: 'select it you want to upload the image to cloud shared Nexus with latest tag.', defaultValue: true)
        // Docker
        booleanParam(name: 'update_image_latest', description: 'Select it if you want to update the image the latest version on local docker.', defaultValue: true)
        booleanParam(name: 'remove_image', description: 'Select it if you want to remove the image from local docker.', defaultValue: true)
        // ECR parameters
        booleanParam(name: 'upload_to_ecr', description: 'select it you want to upload the image to ECR', defaultValue: false)
        choice(choices: ['cloud-dev', 'cloud', 'k2view_shared'], description: 'chose the target ecr', name: 'target')
        string(name: 'ecrImagetag', description: 'Image name to be set', defaultValue: 'k2v-agent:1.0')
        // GCR parameters
        booleanParam(name: 'upload_to_gcr', description: 'select it you want to upload the image to GCR', defaultValue: false)
        // ACR parameters
        booleanParam(name: 'upload_to_acr', description: 'select it you want to upload the image to ACR', defaultValue: false)
    }
    environment {
       WS_APIKEY = credentials('whitesource-apikey')
       WS_WSS_URL = "https://saas.whitesourcesoftware.com/agent"
       WS_USERKEY = credentials('whitesource-serviceaccount-userkey')
   }
    options {
        disableConcurrentBuilds()
    }
    stages{
        stage('pull selected branch') {
            steps {
                dir('k2v-agent') {
                    script{
                        checkout(
                            [$class: 'GitSCM', 
                            branches: [[name: "${repo_branch}"]],
                            doGenerateSubmoduleConfigurations: false, 
                            extensions: [
                                [$class: 'CheckoutOption', timeout: 120],
                                [$class: 'CloneOption', noTags: false, reference: '', shallow: false, timeout: 120]
                            ], 
                            submoduleCfg: [],
                            userRemoteConfigs: [[
                                url: "${repo_link}"]]
                            ]
                        )
                    }
                }
            }
        }
        stage('whitesource code scan') {
            stages {
                stage('update whitesource config') {
                    steps {
                        script {
                            // Split by '/' and take second part
                            def imageNamePart = params.imageName.split("/")[1]
                            // Add projectName
                            def projectName = "k2v-agent-codeSecurityScan-build-#${env.BUILD_NUMBER}"
                            // Update apiKey, productToken and projectName
                            withCredentials([string(credentialsId: 'whitesource-apikey-image-scan-' + imageNamePart, variable: 'LOCAL_PRODUCT_TOKEN')]) {
                                sh "sed -i 's|#productToken=.*|productToken=${LOCAL_PRODUCT_TOKEN}|' ./k2v-agent/whitesource/code-scan.config"
                            }
                            sh "sed -i 's|#apiKey=.*|apiKey=$WS_APIKEY|' ./k2v-agent/whitesource/code-scan.config"
                            sh "sed -i 's|#projectName=.*|projectName=${projectName}|' ./k2v-agent/whitesource/code-scan.config"
                        }
                    }
                }
                stage('Download agent') {
                    steps {
                        sh 'curl -LJO https://unified-agent.s3.amazonaws.com/wss-unified-agent.jar'    
                    } 
                }
                stage('Run agent') {
                    steps {
                        script {
                            // Run the agent and save its output to a file
                            sh 'java -jar wss-unified-agent.jar -c ./k2v-agent/whitesource/code-scan.config -d ./k2v-agent/ > agentOutput.txt'
                            // Read the content of the file
                            def agentOutput = readFile('agentOutput.txt').trim()
                            def scanURL = agentOutput =~ /https:\/\/saas.whitesourcesoftware.com\/Wss[^\[]*/
                            if (scanURL) {
                                scan_link = scanURL[0]
                                echo "Scan Link: ${scan_link}"
                            }        
                        } 
                    }
                }
                stage('Delete agent') {
                    steps {
                        sh 'rm wss-unified-agent.jar'
                    }
                }
            }
        }
        stage('Docker: Building k2v-agent Image'){
            steps{
                dir('k2v-agent') {
                    // Build full size image
                    sh(script : "docker build -t ${params.imageName}:${params.version}_full_size .", returnStdout: false)
                    // Squash the image (Reduse the size of the image)
                    sh(script : "/var/lib/jenkins/.local/bin/docker-squash -v -t ${params.imageName}:${params.version}  ${params.imageName}:${params.version}_full_size", returnStdout: false)
                    // Remove full size image
                    sh(script : "docker rmi -f ${params.imageName}:${params.version}_full_size", returnStdout: false) 
                }            
            }
        }
        stage('Running mend.io image scan pipeline'){
            steps{
                build job: 'mend.io_image_scan',
                    parameters: [
                        string(name: 'image', value: "$params.imageName"),
                        string(name: 'tag', value: "$params.version")
                    ], wait: true
            }
        }
        stage('upload to nexus'){
            when{
                expression { return params.upload_to_nexus }
            }
            steps {
                script {
                    // Login to Nexus
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'nexus_docker_creds', usernameVariable: 'user', passwordVariable: 'pass'], ]){
                        sh "docker login -u $user -p $pass ${nexusDestination}"
                    }
                    // Create tag to push to Nexus 
                    sh "docker tag ${params.imageName}:${params.version} ${nexusDestination}/${params.imageName}:${params.version}"
                    // Push to Nexus
                    sh "docker push ${nexusDestination}/${params.imageName}:${params.version}"
                    // Remove tag related to Nexus
                    sh(script : "docker rmi -f ${nexusDestination}/${params.imageName}:${params.version}", returnStdout: false)
                }
            }
        }
        stage('upload latest to nexus'){
            when{
                expression { return params.upload_to_nexus_as_latest }
            }
            steps {
                script {
                    // Login to Nexus
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'nexus_docker_creds', usernameVariable: 'user', passwordVariable: 'pass'], ]){
                        sh "docker login -u $user -p $pass ${nexusDestination}"
                    }
                    // Create tag to push to Nexus 
                    sh "docker tag ${params.imageName}:${params.version} ${nexusDestination}/${params.imageName}:latest"
                    // Push to Nexus
                    sh "docker push ${nexusDestination}/${params.imageName}:latest"
                    // Remove tag related to Nexus
                    sh(script : "docker rmi -f ${nexusDestination}/${params.imageName}:latest", returnStdout: false)
                }
            }
        }
        stage('upload to cloud nexus'){
            when{
                expression { return params.upload_to_cloud_nexus }
            }
            steps {
                script {
                    // Login to Nexus
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'k2view-share_nexus_docker_creds', usernameVariable: 'user', passwordVariable: 'pass'], ]){
                        sh "docker login -u $user -p $pass ${nexusDestinationCloud}"
                    }
                    // Create tag to push to Nexus 
                    sh "docker tag ${params.imageName}:${params.version} ${nexusDestinationCloud}/${params.imageName}:${params.version}"
                    // Push to Nexus
                    sh "docker push ${nexusDestinationCloud}/${params.imageName}:${params.version}"
                    // Remove tag related to Nexus
                    sh(script : "docker rmi -f ${nexusDestinationCloud}/${params.imageName}:${params.version}", returnStdout: false)
                }
            }
        }
        stage('upload latest to cloud nexus'){
            when{
                expression { return params.upload_to_cloud_nexus_as_latest }
            }
            steps {
                script {
                    // Login to Nexus
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId:'k2view-share_nexus_docker_creds', usernameVariable: 'user', passwordVariable: 'pass'], ]){
                        sh "docker login -u $user -p $pass ${nexusDestinationCloud}"
                    }
                    // Create tag to push to Nexus 
                    sh "docker tag ${params.imageName}:${params.version} ${nexusDestinationCloud}/${params.imageName}:latest"
                    // Push to Nexus
                    sh "docker push ${nexusDestinationCloud}/${params.imageName}:latest"
                    // Remove tag related to Nexus
                    sh(script : "docker rmi -f ${nexusDestinationCloud}/${params.imageName}:latest", returnStdout: false)
                }
            }
        }
        stage('Update image:latest on docker'){
            when{
                expression { return params.update_image_latest }
            }
            steps {
                script {
                   sh "docker tag ${params.imageName}:${params.version} ${params.imageName}:latest"
                }
            }
        } 
        stage('Remove image from docker'){
            when{
                expression { return params.remove_image }
            }
            steps {
                script {
                   sh(script : "docker rmi -f ${params.imageName}:${params.version}", returnStdout: false)
                }
            }
        }
    }

    post {
        success {
            sendEmail(scan_link, "SUCCESS")
        }
        failure {
            script {
                scan_link = "Scan failed"
            }
            sendEmail(scan_link, "FAILED") 
        }
        always {
            deleteDir()
        }
    }
}

def sendEmail(scanLink,status){
    int count=0
    while(count<6){
        try {
            _sendEmail(scanLink,status)
            return
        }catch(Exception ex) {
            sleep(20)
            count++
        }
    }
    println "Email sending failed"
}

def _sendEmail(scanLink,status) {
    mail (
        to: "${params.emailList}",
        subject: "${env.JOB_NAME} job number #${env.BUILD_NUMBER} finished with status - " + status ,
        body: "For getting more information about the scan results visit:\n " + "$scanLink" + "\n\nK2view DevOps Team."
    )
}