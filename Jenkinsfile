def repo_link                   = "https://github.com/k2view-LTD/k2-agent.git"
def repo_branch                 = "master"

pipeline {
    agent {label 'master'}
    parameters {
        // Image
        string(name: 'imageName', description: 'Image name to be set', defaultValue: 'k2view/k2v-agent')
        string(name: 'version', description: 'Image version to be set', defaultValue: '1.0')
        // Docker
        booleanParam(name: 'update_image_latest', description: 'Select it if you want to update the image the latest version on local docker.', defaultValue: true)
        booleanParam(name: 'remove_image', description: 'Select it if you want to remove the image from local docker.', defaultValue: true)
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