pipeline {
  agent { label 'tool' }  // чтобы выполнялось на твоём Jenkins Node

  parameters {
    choice(name: 'ACTION', choices: ['create_or_update', 'delete'], description: 'Что сделать со stack')
    string(name: 'STACK_NAME', defaultValue: 'andrey-heat-stack', description: 'Имя Heat stack')
    string(name: 'SERVER_NAME', defaultValue: 'andrey-heat-vm', description: 'Имя создаваемой VM')
    string(name: 'NET_ID', defaultValue: '17eae9b6-2168-4a07-a0d3-66d5ad2a9f0e', description: 'UUID сети (sutdents-net)')
    string(name: 'KEY_NAME', defaultValue: 'AndreyIL', description: 'Имя keypair в OpenStack')
    string(name: 'SECURITY_GROUP', defaultValue: 'students-general', description: 'Security group')
  }

  stages {
    stage('Check tools') {
      steps {
        sh '''
          set -e
          which openstack
          openstack --version
        '''
      }
    }

    stage('Create/Update stack') {
      when { expression { params.ACTION == 'create_or_update' } }
      steps {
        sh """
          set -e
          # openrc НЕ храним в Git. Он должен лежать на агенте.
          source /home/ubuntu/students-openrc.sh

          if openstack stack show '${params.STACK_NAME}' >/dev/null 2>&1; then
            echo 'Stack exists -> update'
            openstack stack update -t heat/stack.yaml -e heat/env.yaml \\
              --parameter server_name='${params.SERVER_NAME}' \\
              --parameter net_id='${params.NET_ID}' \\
              --parameter key_name='${params.KEY_NAME}' \\
              --parameter security_group='${params.SECURITY_GROUP}' \\
              '${params.STACK_NAME}'
            openstack stack wait '${params.STACK_NAME}'
          else
            echo 'Creating stack'
            openstack stack create -t heat/stack.yaml -e heat/env.yaml \\
              --parameter server_name='${params.SERVER_NAME}' \\
              --parameter net_id='${params.NET_ID}' \\
              --parameter key_name='${params.KEY_NAME}' \\
              --parameter security_group='${params.SECURITY_GROUP}' \\
              '${params.STACK_NAME}'
            openstack stack wait '${params.STACK_NAME}'
          fi

          echo 'STACK STATUS:'
          openstack stack show '${params.STACK_NAME}' -f value -c stack_status

          echo 'SERVER IP:'
          openstack stack output show '${params.STACK_NAME}' server_ip -f value
        """
      }
    }

    stage('Delete stack') {
      when { expression { params.ACTION == 'delete' } }
      steps {
        sh """
          set -e
          source /home/ubuntu/students-openrc.sh
          openstack stack delete -y --wait '${params.STACK_NAME}'
          echo 'Deleted'
        """
      }
    }
  }
}
