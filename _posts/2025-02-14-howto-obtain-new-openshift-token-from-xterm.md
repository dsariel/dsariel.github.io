---
layout: post
title:  HOWTO Obtain new Openshift token from Xterm
date:   2025-02-14 09:15:16
description: OpenShift login token expired because tokens are, by default, valid for a limited time (e.g., 24 hours). Follow these steps to obtain a new token and continue working with the cluster. 
tags: OpenShift token xterm CLI text-web-browser lynx
categories: OpenShift
---

# How to Obtain a New Token for OpenShift from xterm

OpenShift login tokens expire because they are, by default, valid for a limited time (e.g., 24 hours). One way to obtain a new token is by SSHing into the controller machine and 
running the [relogin.sh script](https://github.com/dsariel/scripts/blob/master/OpenShift/relogin.sh). See the content below:


# OpenShift Token Refresh

OpenShift login tokens expire because they are, by default, valid for a limited time (e.g., 24 hours). One way to obtain a new token is by SSHing into the controller machine and running the relogin.sh script.

<iframe src="iframe.php?url=https://github.com/dsariel/scripts/blob/master/OpenShift/relogin.sh" width="100%" height="400px"></iframe>



# ------------------------------


<iframe
  src="https://raw.githubusercontent.com/dsariel/scripts/refs/heads/master/OpenShift/relogin.sh"
  style="width:100%; height:300px;"
></iframe>

# ------------------------------

<script>
fetch("https://raw.githubusercontent.com/dsariel/scripts/master/OpenShift/relogin.sh")
    .then(response => response.text())
    .then(data => {
        document.getElementById("script-content").textContent = data;
    });
</script>

<pre id="script-content">Loading...</pre>


# OpenShift Token Refresh


```html
<style>
.gist .gist-file {
    background-color: #a8a2a2 !important;
    color: #f548f2 !important;
}
</style>

<script src="https://gist.github.com/dsariel/a68bdb55784f23b59f64aa6a6929b73d.js"></script>
```

# ------------------------------



```bash
<!-- Embed script from raw GitHub content -->
curl -s https://raw.githubusercontent.com/dsariel/scripts/master/OpenShift/relogin.sh | cat

```

and using a text-based web browser on the remote server if connecting via the web console is not an option. Follow these steps to obtain a new token and continue working with the cluster:

## Steps to Obtain a New Token

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


