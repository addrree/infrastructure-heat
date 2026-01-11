pipeline {
  agent { label 'tool' }

  parameters {
    choice(name: 'ACTION', choices: ['create_or_update', 'delete'], description: 'Действие со стеком')
    string(name: 'STACK_NAME', defaultValue: 'andrey-heat-stack', description: 'Имя Heat stack')
    string(name: 'SERVER_NAME', defaultValue: 'andrey-heat-vm', description: 'Имя VM внутри stack')
    string(name: 'NET_ID', defaultValue: '17eae9b6-2168-4a07-a0d3-66d5ad2a9f0e', description: 'UUID сети')
    string(name: 'KEY_NAME', defaultValue: 'AndreyIL', description: 'Имя keypair в OpenStack')
    string(name: 'SECURITY_GROUP', defaultValue: 'students-general', description: 'Security group')
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

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
          bash -lc '
            set -e
            . /home/ubuntu/openrc-jenkins.sh

            openstack token issue >/dev/null

            if openstack stack show "${params.STACK_NAME}" >/dev/null 2>&1; then
              echo "Stack exists -> update"
              openstack stack update -t heat/stack.yaml -e heat/env.yaml \\
                --parameter server_name="${params.SERVER_NAME}" \\
                --parameter net_id="${params.NET_ID}" \\
                --parameter key_name="${params.KEY_NAME}" \\
                --parameter security_group="${params.SECURITY_GROUP}" \\
                "${params.STACK_NAME}"
              echo "Waiting for stack to finish..."
                for i in $(seq 1 60); do
                    status=$(openstack stack show "${params.STACK_NAME}" -f value -c stack_status)
                    echo "Status: $status"
                    case "$status" in
                        *_COMPLETE) break ;;
                        *_FAILED) echo "Stack failed"; openstack stack failures list "${params.STACK_NAME}" || true; exit 1 ;;
                    esac
                    sleep 10
                done
            else
              echo "Creating stack"
              openstack stack create -t heat/stack.yaml -e heat/env.yaml \\
                --parameter server_name="${params.SERVER_NAME}" \\
                --parameter net_id="${params.NET_ID}" \\
                --parameter key_name="${params.KEY_NAME}" \\
                --parameter security_group="${params.SECURITY_GROUP}" \\
                "${params.STACK_NAME}"
              echo "Waiting for stack to finish..."
                for i in $(seq 1 60); do
                    status=$(openstack stack show "${params.STACK_NAME}" -f value -c stack_status)
                    echo "Status: $status"
                    case "$status" in
                        *_COMPLETE) break ;;
                        *_FAILED) echo "Stack failed"; openstack stack failures list "${params.STACK_NAME}" || true; exit 1 ;;
                    esac
                    sleep 10
                done
            fi

            echo "STACK STATUS:"
            openstack stack show "${params.STACK_NAME}" -f value -c stack_status

            echo "SERVER IP:"
            openstack stack output show "${params.STACK_NAME}" server_ip -f value
          '
        """
      }
    }

    stage('Delete stack') {
      when { expression { params.ACTION == 'delete' } }
      steps {
        sh """
          set -e
          bash -lc '
            set -e
            . /home/ubuntu/openrc-jenkins.sh
            openstack stack delete -y --wait "${params.STACK_NAME}"
            echo "Deleted: ${params.STACK_NAME}"
          '
        """
      }
    }
  }

  post {
    success { echo "Pipeline SUCCESS" }
    failure { echo "Pipeline FAILED" }
  }
}
