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
    def response = httpRequest "http://httpbin.org/response-headers"
    getPRInfo(org, repo, prId)
}

def getPRInfo(String org, String repo, String prId) {
    String url = "https://api.github.com/repos/${org}/${repo}/pulls/${prId}"

    def response = httpRequest authenticate: 'GITHUB_USER', httpMode: 'GET',
            validResponseCodes: '200',
            url: url
    println("Status: ${response.status}")
    println("Response: ${response.content}")
}