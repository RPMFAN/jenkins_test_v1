pipeline {
    agent any
    environment {
        BUILD = "${env.BUILD_ID}"
    }
    parameters {
        string(name: 'new_branch', defaultValue: 'jenkins_test_v1', description: 'New branch')
        choice(name: 'repository_branch', choices: ['master', 'jenkins_feature_1', 'jenkins_feature_2'], description: 'Choice branch')
        string(name: 'repository_url', defaultValue: 'git@github.com:RPMFAN/jenkins_test_v1.git', description: 'Github repository url')
        booleanParam(name: 'remove', defaultValue: true, description: 'Remove package')
    }
    stages {
        stage('Clone repository') {
            steps {
                    git url: "${params.repository_url}", branch: "${params.repository_branch}"
            }
        }
        stage('Checking repository'){
            steps {
                sh "ls -l"
            }
        }
        stage('Create Branch'){
            steps {
                sh "git checkout -b ${params.branch_new}"
                sh "git branch"
                sh "git checkout ${params.branch_new}"
            }
        }
        stage('Packing project') {
            steps {
                sh '''
                     tar -zcvf /tmp/$BUILD.tar.gz  ./
                     cp /tmp/$BUILD.tar.gz  ./
                '''
            }
        }
        stage('Remove') {
            when {
                expression {params.remove == true}
            }
            steps {
                sh 'rm -f /tmp/*.tar.gz'
            }
        }
        stage('Packing test') {
            steps {
                sh "ls -l"
            }
        }
        stage('Git Push'){
            steps{
                sh """
                git add *
                git commit -a -m 'jenkins_branch'
                git push origin ${params.branch_new} --force
                """
            }
        }
        stage('Test') {
            steps {
                deleteDir()
                git url: "${params.repository_url}"
                sh "git checkout ${params.branch_new}"
                sh "echo 'Remote and Local branches are:'"
                sh 'git branch -a'
                sh "echo 'Files in remote ${params.branch_new}' branch:"
                sh 'ls -l'
                sh """
                if [[ -f \$BUILD.tar.gz ]]; then
                    echo 'All is OK!'
                else
                    echo 'Package in ${params.branch_new} branch not found!' 2>&1
                fi
                """
            }
        }
        stage('Delete branch') {
            steps {
                    sh "git checkout master && git branch -D ${params.branch_new}"
                    sh "git push origin --delete ${params.branch_new}"
            }
        }
    }
    post {
        aborted {
            slackSend (color: '#a5a4a4', message: "ABORTED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        success {
            slackSend (color: '#00c63b', message: "SUCCESSFUL: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
        failure {
            slackSend (color: '#e20606', message: "FAILED: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]' (${env.BUILD_URL})")
        }
    }
        
}
