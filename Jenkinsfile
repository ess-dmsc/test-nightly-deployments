@Library('ecdc-pipeline')
import ecdcpipeline.ContainerBuildNode
import ecdcpipeline.PipelineBuilder

project = "nightly-builds"

container_build_nodes = [
  'ubuntu': new ContainerBuildNode('dockerregistry.esss.dk/ecdc_group/build-node-images/ubuntu22.04-build-node:4.0.0', 'bash -e')
]

// Define number of old builds to keep.
num_artifacts_to_keep = '1'


// Initialize the pipeline builder
pipeline_builder = new PipelineBuilder(this, container_build_nodes)

// Define build stages
builders = pipeline_builder.createBuilders { container ->
  pipeline_builder.stage("${container.key}: Checkout") {
    dir(pipeline_builder.project) {
      scm_vars = checkout scm
    }
    container.copyTo(pipeline_builder.project, pipeline_builder.project)
  }  // stage

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

  pipeline_builder.stage("${container.key}: Ansible Deployment") {
    container.sh """
      which ansible
      ansible --version
      ansible-playbook -i ${pipeline_builder.project}/inventories/site \
      -l efu0234 ${pipeline_builder.project}/efu.yml
    """
  } // stage
}  // createBuilders

builders.each { builder ->
  builder.skipUpload()
}

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
