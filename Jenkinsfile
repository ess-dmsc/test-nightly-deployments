@Library('ecdc-pipeline')
import ecdcpipeline.ContainerBuildNode
import ecdcpipeline.PipelineBuilder

project = "nightly-builds"

container_build_nodes = [
  'ubuntu2204': ContainerBuildNode.getDefaultContainerBuildNode('ubuntu2204')
]

// Define number of old builds to keep.
num_artifacts_to_keep = '1'

// Set number of old builds to keep.
properties([[
  $class: 'BuildDiscarderProperty',
  strategy: [
    $class: 'LogRotator',
    artifactDaysToKeepStr: '',
    artifactNumToKeepStr: num_artifacts_to_keep,
    daysToKeepStr: '',
    numToKeepStr: num_artifacts_to_keep
  ]
]]);

pipeline_builder = new PipelineBuilder(this, container_build_nodes)

builders = pipeline_builder.createBuilders { container ->

pipeline_builder.stage("${container.key}: Checkout GitLab Repo") {
  withCredentials([
    usernamePassword(credentialsId: 'gitlab-accesstoken-dm-ansible-read-rights', usernameVariable: 'GITLAB_USERNAME', passwordVariable: 'GITLAB_ACCESS_TOKEN')
  ]) {
    dir(pipeline_builder.project) {
      // Git checkout using access token
      container.sh """
        git clone https://$GITLAB_USERNAME:$GITLAB_ACCESS_TOKEN@gitlab.esss.lu.se/ecdc/dm-ansible.git
      """
    }
    container.copyTo(pipeline_builder.project, pipeline_builder.project)
  }
} // stage



  pipeline_builder.stage("${container.key}: Setup Python Environment") {
    container.sh """
      which python3
      python3 --version
      python3 -m venv venv
      . venv/bin/activate
      python3 -m pip install --upgrade pip
      python3 --version
    """
  } // stage

  // Install Ansible using Pip
  pipeline_builder.stage("${container.key}: Install Ansible") {
    container.sh """
      . venv/bin/activate
      python3 -m pip install ansible
    """
  } // stage

// Run Ansible Playbook with ProxyJump
pipeline_builder.stage("${container.key}: Run Ansible Playbook with ProxyJump") {
  withCredentials([
    string(credentialsId: 'ansible-vault-all', variable: 'VAULT_PASSWORD'),
    string(credentialsId: 'ssh4-service-account-user', variable: 'SSH_USER'),
    string(credentialsId: 'ssh4-service-account-key', variable: 'SSH_PASSWORD')
  ]) {
    container.sh """
      . venv/bin/activate
      sshpass -p "$SSH_PASSWORD" ansible-playbook -i ./dm-ansible/inventories/site \
      -l efu0234 ./dm-ansible/efu.yml \
      --vault-password-file <(echo "$VAULT_PASSWORD") \
      --become-user=$SSH_USER \
      --extra-vars "ansible_become_pass=$SSH_PASSWORD" \
      --ssh-common-args="-o ProxyJump=$SSH_USER@ssh4.esss.dk"
    """
  }
} // stage

  
}  // createBuilders


// Execute the pipeline
node {
  dir("${project}") {
    scm_vars = checkout scm
  }

  try {
    parallel builders
  } catch (e) {
    throw e
  }

  // Clean up the workspace when the build is complete
  cleanWs()
}
