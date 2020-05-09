# aws-codepipeline-eks-helm

## Usage
- buildspec.yml file, change the path to the Dockerfile on line 21, the Helm release name and the path to the Helm files on line 27.
-  the pre_build step, a kube-config file is copied to be the in the ~/.kube/config, thus commit it at the root of the repository under the name `kube-<ENV>`.
```
apiVersion: v1
clusters:
- cluster:
    server: <EKS_SERVER_URL>
    certificate-authority-data: <CLUSTER_CERTIFICATE_AUTHORITY_DATA>
  name: my-cluster
contexts:
- context:
    cluster: my-cluster
    user: my-user
  name: my-context
current-context: my-context
kind: Config
preferences: {}
users:
- name: my-user
  user:
    exec:
      apiVersion: client.authentication.k8s.io/v1alpha1
      command: aws-iam-authenticator
      args:
        - "token"
        - "-i"
        - "<CLUSTER_NAME>"
```
- Create the pipeline using AWS standard Ubuntu image version 1 w/ the following environment variables
```
AWS_ACCOUNT_ID = <YOUR_ACCOUNT_ID>
AWS_DEFAULT_REGION = <PROJECT_REGION>
IMAGE_REPO_NAME = <ECR_URL>
ENV = <DEPLOYED_ENV>
```
- The last phase of the deployment is the upgrade of the cluster thanks to the `helm upgrade` command. Thus, the CodeBuild process needs to be able to access the Kubernetes cluster. To do so, modify the `aws-auth` configMap with the command `kubectl edit -n kube-system configmap/aws-auth` and add the following lines below the `mapUsers` key:
```
- userarn: arn:aws:iam::<AWS_ACCOUNT_ID>:role/<ROLE_NAME>
  username: <ROLE_NAME>
  groups:
    - system:masters
```

## Reference
- https://www.padok.fr/en/blog/codepipeline-eks-helm
