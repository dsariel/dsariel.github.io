---
layout: post
title:  HOWTO Obtain new Openshift token from Xterm
date:   2025-02-14 09:15:16
description: OpenShift login token expired because tokens are, by default, valid for a limited time (e.g., 24 hours). Follow these steps to obtain a new token and continue working with the cluster. 
tags: OpenShift token xterm text-web-browser lynx
categories: OpenShift
---

# How to Obtain a New Token for OpenShift from xterm

Suppose your OpenShift login token expired because tokens are, by default, valid for a limited time (e.g., 24 hours). Follow these steps to obtain a new token and continue working with the cluster:

## Steps to Obtain a New Token

1. **Install a Text-Based Browser**\
   Install `lynx`, a command-line browser, to access the token request page:

   ```bash
   sudo yum install lynx -y
   ```

2. **Access the Token Request Page**\
   Use `lynx` to navigate to the token request page:

   ```bash
   lynx https://oauth-openshift.apps.ocp.openstack.lab/oauth/token/request
   ```

3. **Retrieve the ******\`\`****** Password**\
   If you are using the default `kubeadmin` user, retrieve the password from the installation directory:

   ```bash
   cat ~/.kube/kubeadmin-password
   ```

   Use this password to log in as `kubeadmin` in the browser interface.

4. **Generate a New Token**\
   Once logged in, obtain a new token. The login command will look like this:

   ```bash
   oc login --token=sha256~<...> --server=https://api.ocp.openstack.lab:6443
   ```

5. **Retry Your Command**\
   With the new token, re-run the desired command. For example:

   ```bash
   oc -n openstack wait openstackcontrolplane controlplane --for condition=Ready --timeout=60m
   ```

By following these steps, you can quickly regain access to your OpenShift cluster and continue your tasks without disruption.



