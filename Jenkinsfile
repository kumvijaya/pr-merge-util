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

node('') {

    String prUrl = params.prUrl

    stage('PR Check') {
        def prInfo = getPRInfo(prUrl)
        echo "prInfo = ${prInfo}"
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
def getPRInfo(String prUrl) {
    def prInfo = [:]
    String prApiUrl = getPRApiUrl(prUrl)
    def pr = getRequest(prApiUrl)
    prInfo['source'] = pr.head.ref
    prInfo['target'] = pr.base.ref
    prInfo['approvalCount'] = getApprovalCount("${prApiUrl}/reviews")
    String requiredStatusDesc = '<<Provide the check description>>' // Check description, update this as per need.
    prInfo['checkSucceeded'] = isCheckStatusSucceeded(pr.statuses_url, requiredStatusDesc)
    return prInfo
}

/**
* Gets PR API Url
*/
def getPRApiUrl(String prUrl) {
    String org = getOrg(prUrl)
    String repo = getRepo(prUrl)
    String prId = getPRNumber(prUrl)
    return "https://api.github.com/repos/${org}/${repo}/pulls/${prId}"
}

/**
* Gets PR Approval count
*/
def getApprovalCount(String prReviewsUrl) {
    def reviews = getRequest(prReviewsUrl)
    def approvedReviews = reviews.findAll { it.state == "APPROVED" }
    return approvedReviews.size()
}

/**
* Checks PR status check successful as per the given status desc
*/
def isCheckStatusSucceeded(String prStatusesUrl, String statusDesc) {
    def statuses = getRequest(prStatusesUrl)
    def requiredStatus = statuses.find { it.description == statusDesc &&  it.state == 'success'}
    return requiredStatus != null
}

/**
* Invokes the get request 
*/
def getRequest(String requestUrl) {
    def response = httpRequest authentication: 'GITHUB_USER_PASS', httpMode: 'GET',
            validResponseCodes: '200',
            url: requestUrl
    def responseJson = new JsonSlurper().parseText(response.content)
    return responseJson
}

/**
* Gets PR number from github pr url 
*/
def getPRNumber(String prUrl) {
    return prUrl.tokenize('/').last()
}

/**
* Gets the org name property from github pr url 
*/
def getOrg(String prUrl) {
    String urlPart = prUrl.replace("https://github.com/", '')
    return urlPart.tokenize('/').first()
}

/**
* Gets the repo name property from github pr url 
*/
def getRepo(String prUrl) {
    String urlPart = prUrl.replace("https://github.com/", '')
    return urlPart.tokenize('/')[1]
}