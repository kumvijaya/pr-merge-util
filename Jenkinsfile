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

String prUrl = params.prUrl
String requiredStatusDesc = 'CI build successful'

node {
    def prInfo = getPRInfo(org, repo, prId)
    echo "prInfo = ${prInfo}"
}

def getPRInfo(String prUrl) {
    def prInfo = [:]
    String org = getOrg(prUrl)
    String repo = getRepo(prUrl)
    String prId = getPRNumber(prUrl)
    String prApiUrl = "https://api.github.com/repos/${org}/${repo}/pulls/${prId}"
    def pr = getRequest(prApiUrl)
    prInfo['source'] = pr.head.ref
    prInfo['target'] = pr.base.ref
    def reviews = getRequest("${prApiUrl}/reviews")
    def approvedReviews = reviews.findAll { it.state == "APPROVED" }
    prInfo['approvalCount'] = approvedReviews.size()
    def statuses = getRequest(pr.statuses_url)
    def requiredStatus = statuses.find { it.description == requiredStatusDesc &&  it.state == 'success'}
    prInfo['checkSucceeded'] = requiredStatus != null
    return prInfo
}

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