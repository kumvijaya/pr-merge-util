# Pull Request Pipeline

This is to check and merge the given pull request, and peform the CICD on merged branch.
Below checks are done as before merging the PR
- PR is approved with required number of approvers
- PR passes pre-defined checks (Future)
- PR is in mergable state (Means not already merged, doesnt have any conflicts)

Once the PR is merged, It checksout the merged branh, performs the CICD.

Also it adds the PR number to deployment package name. Format: ${PACKAGE_NAME}_pr_${PR_NUMBER}

## Environment Variables
Below environment variables can be provided for job run.
- *PR_MERGE_SLAVE_AGENT_LABEL* : The slave agent label to use (Ex: MacSTANDALONE). Default none.
- *PR_MERGE_APPROVAL_COUNT* : Required number of approvals (Ex: 2). Default 0.
- *PR_MERGE_STATUS_LABELS* : Required statuses to check. Provide as comma seperated if more than one. This is future (Ex: unit-test,lint). Default none.

![jenkins-env-vars](https://github.com/kumvijaya/pr-merge-demo/blob/develop/images/env-vars.png)

## Required Credentials
This expects below github credentials created in Jenkins in Credentials store.
- *Type*: UsernamePassword
- *Username*: Github user id
- *Password*: Github PAT (Personal Access Token, This should have read/write access to repo)
- *ID*: GITHUB_USER_PASS

![jenkins-github-creds](https://github.com/kumvijaya/pr-merge-demo/blob/develop/images/github-creds.png)

# Required Job Parameters
This expects job parameter *prUrl*

![jenkins-job-params](https://github.com/kumvijaya/pr-merge-demo/blob/develop/images/job-params.png)

Provide the valid PR Url here (Ex: https://github.com/kumvijaya/pr-merge-demo/pull/1)


# Future Thoughts

Merger util can be kept as Shared Jenkins pipeline library, so that all the repos can leverage this without the code duplication. 

Refer [this](https://www.jenkins.io/doc/book/pipeline/shared-libraries/) for more details on Jenkins Shared Pipeline library



