pipeline {
    agent none
    environment {
      SOURCE_DIR = '/data/pipeline_demo'
    }
    triggers {
        GenericTrigger (
            causeString: 'Triggered', 
            genericVariables: [
              [key: 'ref', value: '$.ref'],
              [key: 'action', value: '$.action'],
              [key: 'merge_commit', value: '$.pull_request.merge_commit_sha'],
              [key: 'branch', value: '$.workflow_run.head_branch'],
              [key: 'repo', value: '$.repository.name'],
              [key: 'pull_request_title', value: '$.pull_request.title']
            ], 
            printContributedVariables: true, 
            printPostContent: true,
            regexpFilterExpression: 'completed\\sdevelop',
            regexpFilterText: '$action $branch',
            token: 'nova'
        )
    }

    stages {
        stage('Deploy to test'){
            when {
                branch 'master'
            }
            agent {
              node {
                label "dingo_stack"
              }
            }
            steps{
              echo 'pull images to dev'
              sh 'kolla-ansible -i /root/multinode pull --tag nova -e openstack_tag=latest -vvv'
              echo 'deploy images to develop '
              sh 'kolla-ansible -i /root/multinode upgrade --tag nova -e openstack_tag=latest -vvv'
            }
        }
        stage('deploy to dev'){
            when {
                anyOf { branch 'develop'; branch 'stable/2023.2' }
            }
            agent {
              node {
                label "dingo_stack"
              }
            }
            steps{
              echo 'pull images to dev'
              sh 'kolla-ansible -i /root/multinode pull --tag nova -e openstack_tag=${branch} --become-user root -vvvv'
              echo 'deploy images to develop '
              sh 'kolla-ansible -i /root/multinode upgrade --tag nova -e openstack_tag=${branch} --become-user root -vvvv'
            }
        }
    }
}
