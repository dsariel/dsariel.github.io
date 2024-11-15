---
layout: post
title:  HOWTO develop an OpenShift application
date:   2024-11-11 16:40:16
description: OpenShift container runtime poses restrictions, and simple application containerization will not work in the vast majority of cases. In this post, steps are described to download CRC locally, containerize an application, apply necessary adjustments to overcome the restrictions, and push the resulting tested image to a registry, from which a production OpenShift instance will pull it. 
tags: OpenShift crc application development CodeReadyContainers
categories: sample-posts
---

# How to install Red Hat OpenShift Local on your laptop
Since OpenShift's container runtime poses constraints, one can't simply create a `Containerfile` (or `Dockerfile`) and test a containerized application with Podman (or Docker), expecting it to be flawlessly deployed and work on OpenShift just because it worked on the Podman (or Docker) runtime.

One possible approach (subject to limitations described below) is installing an all-in-one version of OpenShift and proceeding with the deployment, testing, and debugging cycles on the local machine.

Using a local instance of OpenShift can save a considerable amount of time by eliminating the step of pushing large container images to a container registry. This allows you to create a container image locally and perform an OpenShift application deployment directly from that local image. However, this all-in-one version of OpenShift has limitations compared to a full-fledged OpenShift deployment across multiple nodes with dedicated hardware. For more details, see the [Red Hat CodeReady Containers documentation](https://docs.redhat.com/en/documentation/red_hat_codeready_containers/2.0/html/getting_started_guide/introducing-codeready-containers_gsg#differences-from-production-openshift-install_gsg).

If your application requires features that are not supported by the all-in-one version, this method may not work for those applications (or may not test all the features of the application). Nevertheless, it can be useful for an initial kickstart and a subset of features.

With that said, follow the steps to deploy CRC locally as outlined in the [Red Hat OpenShift Local documentation](https://docs.redhat.com/en/documentation/red_hat_codeready_containers/2.0/html/getting_started_guide/index). I have choosen to authorize RedHat SSO: 

```
"Red Hat SSO by rh-sso wants to access your github_user account" 
```

and after filling a short form with details required to create a RedHat account, I was able to download the latest version of the OpenShift Local along with `pull secret`. 





# Understanding Security Context Constraints (SCCs) in OpenShift

In OpenShift, Security Context Constraints (SCCs) play a crucial role in controlling the permissions of containers and maintaining the security of the cluster. To grasp the significance of SCCs, it’s essential to understand how containers interact with protected Linux functions.

## What Is a Security Context Constraint?

When a container runs a process, it inherently restricts that process from accessing certain privileged Linux functionalities. These limitations include actions such as:

- Accessing shared file systems
- Running as a privileged container or as root
- Executing specific Linux commands, such as `kill`

These restrictions are in place to maintain container isolation. If processes within containers had unrestricted access, they could interfere with other processes and potentially compromise the isolation and security of the system.

However, there are situations where applications require access to some of these protected functions. This is where Security Contexts and Security Context Constraints come into play.

## How Security Contexts Work

A Security Context defines the security privileges and access controls applied to a pod or container. When an application needs access to certain protected Linux functions, the pod running the container must be configured with a security context that specifies the required access.

Key components of the configuration include:

- The pod’s security context which details the access permissions needed.
- A service account associated with the pod that authorizes these permissions.

## The SCC Authorization Process

The process for granting these permissions involves several steps:

1. **Developer creates an application**: The developer builds an application that requires access to certain protected functions.
2. **Deployer writes a deployment manifest**: This manifest specifies the security context and the service account needed for the application.
3. **Association with an SCC**: The deployment is linked to an SCC, either predefined by OpenShift or custom-created by the cluster administrator.
4. **Cluster administrator control**: The SCC is created and managed by the cluster administrator, who has full control over which pods can access which functionalities.

## How Pods Are Authorized

When the deployment manifest is applied:

- The pod specifies the service account to be used.
- The service account is associated with an SCC, directly or through RBAC (Role-Based Access Control).
- Admission control checks if the pod’s security context aligns with the permissions defined in the SCC. If the request is valid, the pod is allowed to run with the specified access; otherwise, it is denied.

## Example: Pod Manifest and Deployment

In a typical deployment manifest, security contexts can be set at two levels:

- **Container-level security context**: Specifies access permissions for an individual container.
- **Pod-level security context**: Applies permissions to all containers within the pod.

The pod manifest also includes the service account name used to authorize the specified access.

## Structure of Security Context Constraints

SCCs can either be predefined or custom-made by cluster administrators. Here’s a comparison between a restricted SCC and a custom SCC:

- **Restricted SCC**: This built-in SCC in OpenShift enforces strict limitations. It drops most capabilities and restricts user and group permissions, ensuring minimal access beyond default settings.
- **Custom SCC**: Administrators can create tailored SCCs that provide specific permissions, such as allowing a user ID range (e.g., 1000-2000) or granting additional capabilities.


```yaml
containers:
  - resources: {}
    terminationMessagePath: /dev/termination-log
    name: demo
    command:
      - tox
      - -v
      - -e dev
    securityContext:
      capabilities:
        drop:
          - ALL
      runAsUser: 1234
      runAsGroup: 5678
      runAsNonRoot: true
      allowPrivilegeEscalation: false
```

Let me know if you need any additional formatting or edits!

## Conclusion

SCCs are essential for maintaining a secure and controlled environment in OpenShift. While containers by default limit access to protect system integrity, SCCs provide the flexibility needed when applications require elevated permissions. This balance ensures that cluster administrators can grant necessary permissions while maintaining overall security.
