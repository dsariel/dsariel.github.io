---
layout: post
title: How to Obtain a New OpenShift Token and relogin
date: 2025-02-14 09:15:16
description: An OpenShift login token expired because tokens are, by default, valid for a limited time. Follow these steps to obtain a new token and continue working with the cluster.
tags: OpenShift token xterm CLI text-web-browser lynx web-browser
categories: OpenShift
---

# General
OpenShift login tokens expire because they are, by default, valid for a limited time (e.g., 24 hours). There are several ways to re-login.


## Obtain a token from a web console
If you have access to the web console and re-login from there:
```bash
oc whoami --show-console
```


## Run a re-login script

If connecting via the web console is not an option, ssh into the controller machine and run the
[relogin.sh script](https://github.com/dsariel/scripts/blob/master/OpenShift/relogin.sh):

<script>
fetch("https://raw.githubusercontent.com/dsariel/scripts/master/OpenShift/relogin.sh")
    .then(response => response.text())
    .then(data => {
        document.getElementById("script-content").textContent = data;
    });
</script>

<pre id="script-content">Loading...</pre>



## Steps to Obtain a New Token using command line browser
Another (more laborious) way is using a text-based web browser on the remote server (again, if the web console is not an option).
Follow these steps to obtain a new token and continue working with the cluster:

1. **Install a Text-Based Browser**\
   Install `lynx`, a command-line browser, to access the token request page:

   ```bash
   sudo yum install lynx -y
   ```

2. **Retrieve a password for the `kubeadmin` user**\
   If you are using the default `kubeadmin` user, retrieve the password from the installation directory:

   ```bash
   cat ~/.kube/kubeadmin-password
   ```

   Use this password to log in as `kubeadmin` in the browser interface during the next stage.

3. **Access the Token Request Page**\
   Use `lynx` to navigate to the token request page:

   ```bash
   lynx https://oauth-openshift.apps.ocp.openstack.lab/oauth/token/request
   ```

4. **Obtain a New Token**\
   Once logged in, obtain a new token. The login command will look like this:

   ```bash
   oc login --token=sha256~<...> --server=https://api.ocp.openstack.lab:6443
   ```
