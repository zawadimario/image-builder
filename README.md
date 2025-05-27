### Simple steps to create an image builder.

1. Generare a GitHub token sufficient to clone a repository
2. Generate a DockerHub token with sufficient permissions to push images
3. Create a dockerconfigjson to be used by the builder, also required by the kubernetes secret mount in the pod.

#### Generate dockerconfig.json
Replace DOCKER_USERNAME and DOCKER_PASSWORD with your DockerHub username and personal access token (from Docker Hub > Account Settings > Security > New Access Token) and use that as the password. It’s more secure and easier to rotate.

```
echo '{"auths":{"https://index.docker.io/v1/":{"auth":"'"$(echo -n '<DOCKER_USERNAME>:<DOCKER_PASSWORD>' | base64)"'"}}}' > dockerconfig.json
```
#### Create a kubernetes secret from the dockerconfig.json in the previous step

```
kubectl create secret generic regcred --from-file=config.json=dockerconfig.json
```

#### Determine the repository to use and the Dockerfile path/context

This is to allow Kaniko to find the context where to build the image.

For instance, if the Dockerfile is in the project root directory, you may just provide `Dockerfile` as the context. This value is to be provided against the key `--context=` in the builder container arguments.

#### Modify the builder Kubernetes Job spec
A job spec is prefered since it will execute the task and be automatically purged once the image is pushed.

Provide the correct image path to the key `--destination` for the builder to push the image.

#### Acknowledgements
For more details and up-to-date code visit the [Official Kaniko GitHub](https://github.com/GoogleContainerTools/kaniko?tab=readme-ov-file#running-kaniko-in-a-kubernetes-cluster) repository.