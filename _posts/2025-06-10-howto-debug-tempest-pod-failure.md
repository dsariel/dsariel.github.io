---
layout: post
title: "Debugging OpenStack Tempest Pods in OpenShift Series: Failure to Import a 997MB qcow2 Image"
date: 2025-06-10 19:15:00 +0300
description: "The issue was with a 997MB qcow2 image that would not import properly. Here's how I solved this step by step."
tags: [OpenShift, tempest pod]
categories: [OpenShift]
---


I was running OpenStack Tempest tests in my OpenShift cluster when I hit a problem. The pod `tempest-tests-tempest-workflow-step-00-multi-thread-testing` failed during image import. The issue was with a 997MB qcow2 image that would not import properly. Here's how I solved this step by step.

## The Problem

My Tempest pod was failing during the image creation phase. The logs showed it was stuck trying to import a 997MB qcow2 image, and it kept timing out after 300 seconds. This is a common problem when working with large images.

```bash
Current status: importing. Waiting for image to become active...
```

This message was just looping forever until the pod gave up and died.

## Step 1: You Cannot Exec Into a Failed Pod

First thing - you cannot exec into a failed pod:

```bash
oc exec -it tempest-tests-tempest-workflow-step-00-multi-thread-testing -- /bin/bash
```

And got this error message:
```
error: cannot exec into a container in a completed pod; current phase is Failed
```

This makes sense. Failed pods do not respond to commands.

## Step 2: Check the Pod Logs and Events

Before doing anything else, I needed to understand what happened:

```bash
# Get the pod logs
oc logs tempest-tests-tempest-workflow-step-00-multi-thread-testing

# Get detailed information about the pod
oc describe pod tempest-tests-tempest-workflow-step-00-multi-thread-testing

# Check events related to this pod
oc get events --field-selector involvedObject.name=tempest-tests-tempest-workflow-step-00-multi-thread-testing
```

The logs showed me that the script was downloading a nearly 1GB image and then trying to import it into OpenStack Glance, but it was timing out during the import phase.

## Step 3: Handle OpenShift Security Policies

Now I needed to create a debug pod. OpenShift has strict security policies. I tried the simple approach:

```bash
oc run debug-tempest --image=tempest-tests-tempest-workflow-step-00-multi-thread-testing --rm -it -- /bin/bash
```

And got multiple security violations:
```
Warning: would violate PodSecurity "restricted:latest": allowPrivilegeEscalation != false...
```

I needed to follow OpenShift's security requirements.

## Step 4: Create a Security-Compliant Debug Pod

I created a proper YAML manifest that meets OpenShift's security policies:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: debug-tempest
spec:
  restartPolicy: Never
  securityContext:
    runAsNonRoot: true
    runAsUser: 1001
    runAsGroup: 1001
    fsGroup: 1001
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: debug-tempest
    image: YOUR_ACTUAL_IMAGE_HERE  # Get this from the failed pod
    command: ["/bin/bash"]
    args: ["-c", "sleep 3600"]
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
      runAsNonRoot: true
      runAsUser: 1001
      runAsGroup: 1001
      seccompProfile:
        type: RuntimeDefault
    env:
    - name: HOME
      value: /tmp
    - name: OS_CLOUD
      value: default
    volumeMounts:
    - name: temp-dir
      mountPath: /tmp
    - name: var-lib-tempest
      mountPath: /var/lib/tempest
  volumes:
  - name: temp-dir
    emptyDir: {}
  - name: var-lib-tempest
    emptyDir: {}
```

To get the actual image name, I ran:
```bash
oc get job tempest-tests-tempest-workflow-step-00-multi-thread-testing -o jsonpath='{.spec.template.spec.containers[0].image}'
```

## Step 5: Alternative One-Line Command

If you prefer not to use YAML files, here is a one-line command:

```bash
oc run debug-tempest --image=$(oc get job tempest-tests-tempest-workflow-step-00-multi-thread-testing -o jsonpath='{.spec.template.spec.containers[0].image}') --rm -it --restart=Never --overrides='
{
  "spec": {
    "securityContext": {
      "runAsNonRoot": true,
      "runAsUser": 1001,
      "seccompProfile": {"type": "RuntimeDefault"}
    },
    "containers": [{
      "name": "debug-tempest",
      "image": "'$(oc get job tempest-tests-tempest-workflow-step-00-multi-thread-testing -o jsonpath='{.spec.template.spec.containers[0].image}')'",
      "command": ["/bin/bash"],
      "securityContext": {
        "allowPrivilegeEscalation": false,
        "capabilities": {"drop": ["ALL"]},
        "runAsNonRoot": true,
        "runAsUser": 1001,
        "seccompProfile": {"type": "RuntimeDefault"}
      }
    }]
  }
}' -- /bin/bash
```

## Step 6: Debug the Actual Issue

Once I got into the debug pod, I could start investigating:

```bash
# Test if the image URL is accessible
curl -I http://a.b.c.d/dfg-network/custom_neutron_guest_rhel_8.4.qcow2

# Check OpenStack connectivity
. openstackrc
openstack image list

# See if the image got partially created
openstack image show 11111111-1111-1111-1111-111111111111
```

## What I Discovered

In my case, I discovered that the **Timeout was too short**. A 997MB image needs more than 5 minutes to import on our storage backend.
Check how much time it takes and add 20% spear time.


```bash
# Manual image upload
time openstack image create \
  --disk-format qcow2 \
  --id 11111111-1111-1111-1111-111111111111 \
  --file custom_neutron_guest_rhel_8.4.qcow2 \
  --public \
  custom_neutron_guest_rhel_8.4
```

## Lessons Learned

1. **OpenShift security is not optional** - Learn to work with it, not against it
2. **Image imports can be slow** - Plan your timeouts accordingly
3. **oc debug is your friend** - When it works with the security context

## Bonus Tip

If you're running into similar issues regularly, consider pre-loading your test images during cluster setup rather than during test execution. Your future self will thank you when tests aren't timing out at 2 AM.

Remember, debugging in OpenShift is like debugging anywhere else, except with more YAML and more security warnings. But hey, at least the error messages are usually pretty clear about what's wrong!

Happy debugging, and may your pods always run to completion!
