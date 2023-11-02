import groovy.json.JsonSlurper

properties([
    durabilityHint('PERFORMANCE_OPTIMIZED'),
    disableResume(),
    [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator',  daysToKeepStr: '90', numToKeepStr: '50']],
    parameters([
        string(
            defaultValue: 'https://github.com/kumvijaya/pr-test/pull/1',
            name: 'prUrl',
            trim: true,
            description: 'Provide github pull request url: (Example: https://github.com/kumvijaya/pr-test/pull/1).'
        )
    ])
])

String agent = getEnvValue('PR_MERGE_SLAVE_AGENT_LABEL', '')

node(agent) {

    String prUrl = params.prUrl

    stage('PR Check') {
        def prInfo = getPRInfo(prUrl)
        echo "prInfo = ${prInfo}"
        boolean mergeable = canMerge(prInfo)
        echo "mergeable = ${mergeable}"
    }

    stage('PR Merge') {
        def prInfo = getPRInfo(prUrl)
        echo "prInfo = ${prInfo}"
    }

    stage('Build') {
        // Build your application
        echo 'your-build-command'
    }

    stage('Test') {
        // Run tests
        echo 'your-test-command'
    }

    stage('Package') {
        // Package your application
        echo 'your-package-command'
    }

    stage('Deploy') {
        // Deploy your application to a target environment
        echo 'your-deploy-command'
    }
}

/**
* Gets the PR Info for the given PR url.
*/
private def getPRInfo(String prUrl) {
    def prInfo = [:]
    String prApiUrl = getPRApiUrl(prUrl)
    def pr = getRequest(prApiUrl)
    prInfo['source'] = pr.head.ref
    prInfo['target'] = pr.base.ref
    prInfo['state'] = pr.state
    prInfo['merged'] = pr.merged
    prInfo['mergeable'] = pr.mergeable
    prInfo['mergeable_state'] = pr.mergeable_state
    prInfo['approvalCount'] = getApprovalCount("${prApiUrl}/reviews")
    prInfo['checkSucceeded'] = isCheckStatusSucceeded(pr.statuses_url)
    return prInfo
}

/**
* Gets PR API Url
*/
private String getPRApiUrl(String prUrl) {
    String org = getOrg(prUrl)
    String repo = getRepo(prUrl)
    String prId = getPRNumber(prUrl)
    return "https://api.github.com/repos/${org}/${repo}/pulls/${prId}"
}

/**
* Gets PR Approval count
*/
private int getApprovalCount(String prReviewsUrl) {
    def reviews = getRequest(prReviewsUrl)
    def approvedReviews = reviews.findAll { it.state == "APPROVED" }
    return approvedReviews.size()
}

/**
* Checks PR status check successful as per the given status desc
*/
private boolean isCheckStatusSucceeded(String prStatusesUrl) {
    String checkDesc = getEnvValue('PR_MERGE_CHECK_DESC', '')
    boolean checkSucceeded = true
    if(!isEmpty(checkDesc)) {
        def statuses = getRequest(prStatusesUrl)
        def requiredStatus = statuses.find { it.description == statusDesc &&  it.state == 'success'}
        checkSucceeded = (requiredStatus != null)
    }
    return checkSucceeded
}

/**
* Invokes the get request 
*/
private def getRequest(String requestUrl) {
    def response = httpRequest authentication: 'GITHUB_USER_PASS', httpMode: 'GET',
            validResponseCodes: '200',
            url: requestUrl
    def responseJson = new JsonSlurper().parseText(response.content)
    return responseJson
}

/**
* Gets PR number from github pr url 
*/
private String getPRNumber(String prUrl) {
    return prUrl.tokenize('/').last()
}

/**
* Gets the org name property from github pr url 
*/
private String getOrg(String prUrl) {
    String urlPart = prUrl.replace("https://github.com/", '')
    return urlPart.tokenize('/').first()
}

/**
* Gets the repo name property from github pr url 
*/
private String getRepo(String prUrl) {
    String urlPart = prUrl.replace("https://github.com/", '')
    return urlPart.tokenize('/')[1]
}

/**
* Check given PR can be merged
*/
private boolean canMerge(prInfo) {
    int approvalcount = Integer.parseInt(getEnvValue('PR_MERGE_APPROVAL_COUNT', '2'))
    echo "Required approvalcount: $approvalcount"
    return (prInfo.approvalCount == approvalcount
        && prInfo.checkSucceeded
        && prInfo.state == 'open'
        && prInfo.mergeable
        && !prInfo.merged
        && prInfo.mergeable_state == 'clean')
}

/**
* Gets the env variable value
*/
private String getEnvValue(String envKey, String defaultValue='') {
	String envValue = defaultValue
	def envMap = env.getEnvironment()
    echo "envMap= $envMap"
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
    return input?.trim()
}