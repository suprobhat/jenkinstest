pipeline {
  agent any
  triggers {
    GenericTrigger(
     genericVariables: [
      //[key: 'pr_id', value: '$.pull_request.id'],
      //[key: 'pr_state', value: '$.pull_request.state'],
      [key: 'repo_git_url', value: '$.pull_request.head.repo.clone_url'],
      [key: 'pr_branch', value: '$.pull_request.head.ref'],
      [key: 'base_branch', value: '$.pull_request.base.ref']
      
     ],

     //causeString: 'Triggered on $pr_id',
     //causeString: 'Triggered on $pr_state',

     token: '12345',
     tokenCredentialId: '',

     printContributedVariables: true,
     printPostContent: true,

     silentResponse: false,

     //regexpFilterText: '$ref',
     //regexpFilterText: '$pr_state',
     //regexpFilterExpression: 'refs/heads/' + any
    )
  }
  stages {
    stage('Some step') {
      steps {
        //sh "echo $pr_id"
        //sh "echo $pr_state"
        sh "echo hello"
        //sh "echo $pr_branch"
        //sh "echo $base_branch"
      }
    }
  }
}
