import groovy.transform.Field

@Field def images = [
    "image1": "../img1",
    "image2": "../img2",
    "image3": "../img3"
]

@Field def requires_libs = [
    "image1",
    "image2"
]

@Field def requires_nvidia_docker = [
    "image3"
]


@Field def dockerfile_args = """
    --runtime=nvidia
"""

def getImageName(String image) {
    image_tag = sh(script: "echo $(pwd) | shasum | cut -c1-6", returnStdout: true).trim()
    return "${image}:${image_tag}"
}

pipeline{
    agent none

    options {
        skipDefaultCheckout true
    }

    stages {
        stage("Clean Workspace") {
            agent {
                docker {
                    image "ubuntu:16.04"
                    args "--user root"
                    label "test"
                }
            }
        }
        stage("Checkout") {
            agent {
                docker {
                    image "ubuntu:16.04"
                    label "test"
                }
            }
            steps {
                script {
                    recursiveCheckout()
                }
            }
        }
        stage("Container Build") {
            agent {
                dockerfile {
                    args "--group-add docker -v /var/run/docker.sock:/var/run/docker.sock"
                    label "test"
                }
            }
            steps {
                script {
                    scm_variables = checkout scm
                    missing_images = loadDockerImages()

                    for (image in images) {
                        image_context = images[image]
                        image_name = getImageName(image)

                        if (image in requires_libs) {
                            sh (script: "echo ${image}")
                        }

                        if (image in requires_nvidia_docker) {
                            sh (script: "echo 'nvidia-docker build -t ${image_name} ${image_context}'")
                        }
                        else {
                            sh (script: "echo 'docker build -t ${image_name} ${image_context}'")
                        }
                    }
                }
            }
        }
    }
}
