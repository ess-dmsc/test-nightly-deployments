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
    dir(pipeline_builder.project) {
      // Git checkout using access token
      container.sh """
        git clone https://any_username:_58bVWB3aXGBENqrzHTK@gitlab.esss.lu.se/ecdc/dm-ansible.git
      """
    }
    container.copyTo(pipeline_builder.project, pipeline_builder.project)
  }  // stage

  pipeline_builder.stage("${container.key}: Install Dependencies") {
    container.sh """
      which python
      python --version
    """
  } // stage

  pipeline_builder.stage("${container.key}: Ansible Deployment") {
    container.sh """
      find . -maxdepth 2 -name

      which ansible
      ansible --version
      ansible-playbook -i ${pipeline_builder.project}/inventories/site \
      -l efu0234 ${pipeline_builder.project}/efu.yml
    """
  } // stage

}  // createBuilders

// Parallel execution of builders
try {
  parallel builders
} catch (e) {
  throw e
}

// Clean workspace after build
cleanWs()
