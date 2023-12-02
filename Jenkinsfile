import groovy.json.JsonSlurper

pipeline {
     agent {
        label "MacSTANDALONE"
    }
    environment {
        // properties([
        //     parameters([
        //         string(
        //             defaultValue: 'https://github.com/kumvijaya/pr-merge-demo/pull/1',
        //             name: 'prUrl',
        //             trim: true,
        //             description: 'Provide GitHub pull request URL: (Example: https://github.com/kumvijaya/pr-merge-demo/pull/1).'
        //         )
        //     ])
        // ])
        def prMergeInfo = processMerge(params.prUrl)
        TARGET_BRANCH = prMergeInfo.target_branch
        REPO_URL = prMergeInfo.pr_repo_url
        PR_NUMBER = prMergeInfo.pr_number
    }
    
    stages {
        stage('Checkout') {
            git branch: env.TARGET_BRANCH, url: env.REPO_URL
        }
        stage('Build') {
            powershell 'npm install'
        }
        stage('Test') {
            powershell 'npm test'
        }
        stage('Deploy') {
            powershell 'npm pack'
            appendPackageWithPRNumber(env.PR_NUMBER)
        }
    }
            
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