import groovy.json.JsonSlurper

properties([
    durabilityHint('PERFORMANCE_OPTIMIZED'),
    disableResume(),
    [$class: 'BuildDiscarderProperty', strategy: [$class: 'LogRotator',  daysToKeepStr: '90', numToKeepStr: '50']],
    parameters([
        string(
            defaultValue: 'kumvijaya',
            name: 'org',
            trim: true,
            description: 'Provide github org name: (Example: kumvijaya).'
        ),
        string(
            defaultValue: 'pr-test',
            name: 'repo',
            trim: true,
            description: 'Provide the repository name (Ex: samples-repo1)'
        ),
        string(
            defaultValue: '1',
            name: 'prId',
            trim: true,
            description: 'Provide the pull request id (Ex: 1)'
        )
    ])
])

String org = params.org
String repo = params.repo
String prId = params.prId

node {
    getPRInfo(org, repo, prId)
}

def getPRInfo(String org, String repo, String prId) {
    String prUrl = "https://api.github.com/repos/${org}/${repo}/pulls/${prId}"
    // def response = httpRequest authentication: 'GITHUB_USER_PASS', httpMode: 'GET',
    //         validResponseCodes: '200',
    //         url: prUrl
    // def prInfo = new JsonSlurper().parseText(response.content)
    def prInfo = getRequest(prUrl)
    echo "prInfo.head.ref: ${prInfo.head.ref}"
    echo "prInfo.base.ref: ${prInfo.base.ref}"
    String prReviewsUrl = "${prUrl}/reviews"
    // response = httpRequest authentication: 'GITHUB_USER_PASS', httpMode: 'GET',
    //     validResponseCodes: '200',
    //     url: prReviewsUrl
    // def reviewInfo = new JsonSlurper().parseText(response.content)
    def reviewInfo = getRequest(prReviewsUrl) 
    echo "prInfo.reviews: ${reviewInfo}"
}

def getRequest(String requestUrl) {
    def response = httpRequest authentication: 'GITHUB_USER_PASS', httpMode: 'GET',
            validResponseCodes: '200',
            url: requestUrl
    def responseJson = new JsonSlurper().parseText(response.content)
    return responseJson
}