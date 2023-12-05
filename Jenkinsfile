import groovy.json.JsonSlurper

pipeline {
    agent {
        label "MacSTANDALONE"
    }
    stages {
        stage ('Build with PR') {
            steps {
                script {
                    setJobProperties()
                    def prMergeInfo = processMerge(params.prUrl)
                    if(prMergeInfo.already_merged || prMergeInfo.merged) {
                        echo "Given pull request ${prMergeInfo.already_merged ? 'found already merged' : 'merged'}, Proceeding CCID on target branch"
                        stage('Checkout') {
                            // Check out your source code repository as per pr target branch
                            git branch: prMergeInfo.target_branch, url: prMergeInfo.pr_repo_url
                        }
                        stage('Build') {
                            // Build your application
                            powershell 'npm install'
                        }
                        stage('Test') {
                            // Run tests
                            powershell 'npm test'
                        }
                        // stage('Deploy') {
                        //     // Deploy your application to a target environment
                        //     powershell 'npm pack'
                        //     appendPackageWithPRNumber(prMergeInfo.pr_number)
                        // }
                    }else {
                        error "Pull request not merged, Please check the PR."
                    }
                }
            }
        }
    }
    post {
        success {
            echo "success!!"
        }
        failure {
            echo "failure!!"
        }
    }
}

/**
 * Sets the properties for the job.
 */
def setJobProperties() {
    properties([
        durabilityHint('PERFORMANCE_OPTIMIZED'),
        disableResume(),
        [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator',  daysToKeepStr: '90', numToKeepStr: '50']],
        parameters([
            string(
                defaultValue: 'https://github.com/kumvijaya/pr-merge-demo/pull/1',
                name: 'prUrl',
                trim: true,
                description: 'Provide GitHub pull request URL: (Example: https://github.com/kumvijaya/pr-merge-demo/pull/1).'
            )
        ])
    ])
}

/**
* Process the given PR
*/
def processMerge(prUrl) {
    def prMergeInfo = [:]
    stage('Merge PR') {
        dir('merge') {
            checkout scm
            withCredentials([usernamePassword(credentialsId: 'GITHUB_USER_PASS', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_PASSWORD')]) {
                sh """
                    python -m pip install -r requirements.txt --user
                    python git-merger.py -p ${prUrl}
                """
                prMergeInfo = readJSON file: 'git_merge_ouput.json'
            }
            deleteDir()
        }
    }
    return prMergeInfo
}

/**
* Appends package name with PR number
*/
private appendPackageWithPRNumber(prNumber) {
    String name = powershell(returnStdout: true, script:'npm pkg get name | xargs echo').trim()
    String version = powershell(returnStdout: true, script:'npm pkg get version | xargs echo').trim()
    sh "mv ${name}-${version}.tgz ${name}-${version}_pr_${prNumber}.tgz"
}