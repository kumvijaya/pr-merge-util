import groovy.json.JsonSlurper

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

//getEnvValue('PR_MERGE_SLAVE_AGENT_LABEL', '')
//MacSTANDALONE

node('abcs') {
    def prInfo = processMerge(params.prUrl)
    echo "prInfo = ${prInfo}"
    if(prInfo.proceed_cicd) {
        stage('Checkout') {
            // Check out your source code repository as per pr target branch
            git branch: prInfo.target_branch, url: prInfo.pr_repo_url
        }
        stage('Build') {
            // Build your application
            powershell 'npm install'
        }
        stage('Test') {
            // Run tests
            powershell 'npm test'
        }
        stage('Deploy') {
            // Deploy your application to a target environment
            powershell 'npm pack'
            appendPackageWithPRNumber(prInfo.pr_number)
        }
        // Additional stages or post-build actions can be added here
    }
}

def processMerge(prUrl) {
    def prInfo = [:]
    stage('Merge PR') {
        dir('merge') {
            checkout scm
            withCredentials([usernamePassword(credentialsId: 'GITHUB_USER_PASS', usernameVariable: 'GITHUB_USER', passwordVariable: 'GITHUB_PASSWORD')]) {
                sh "python -m pip install requests==2.26.0 --user"
                sh "python git-merger.py -p ${params.prUrl}"
                prInfo = readJSON file: 'git_merge_ouput.json'
            }
            deleteDir()
        }
    }
    return prInfo
}

/**
* Gets the env variable value
*/
private String getEnvValue(String envKey, String defaultValue='') {
	String envValue = defaultValue
	def envMap = env.getEnvironment()
	envMap.each{ key, value ->
		if (key == envKey) {
			envValue = !isEmpty(value) ? value : defaultValue
			return envValue
		}
	}
	return envValue
}

/**
* Is given string empty
*/
private boolean isEmpty(String input) {
    return !input?.trim()
}

/**
* Appends package name with PR number
*/
private appendPackageWithPRNumber(prNumber) {
    String name = powershell(returnStdout: true, script:'npm pkg get name | xargs echo').trim()
    String version = powershell(returnStdout: true, script:'npm pkg get version | xargs echo').trim()
    String packageName = "${name}-${version}"
    String fileName = "${packageName}.tgz"
    String newFileName = "${packageName}_pr_${prNumber}.tgz"
    echo "Updating packageName (${fileName}) to ${newFileName}"
    sh "mv ${fileName} ${newFileName}"
    sh "ls -ltr"
}