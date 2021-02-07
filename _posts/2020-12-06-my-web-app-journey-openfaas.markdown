---
layout: post
title: My Web App Journey - OpenFaaS
date: 2020-12-06 01:31:39.000000000 +00:00
type: post
parent_id: '0'
published: true
password: ''
status: publish
categories:
- Programming
- Technology
tags:
- Kubernetes
- My Web App Journey
- OpenFaaS
author: Jonathan Tweedle
permalink: "/2020/12/06/my-web-app-journey-openfaas/"
---
I feel like this is more of a step backwards. I mean in the [last post][previous_post] I was starting to build some interfaces and about to start with the persistance layer.

I purchased the <a href="https://www.khadas.com/vim3l">Khadas Vim3L</a> in  the hopes of building an HTPC for my wife but sadly it did not work with the streaming media sites due to DRM issues. The device is basically a nice powerful Raspberry Pi with an added NPU which could be used for some machine learning down the line. So I repurposed the device in order to try my hand at using <a href="https://www.openfaas.com/">OpenFaaS</a>, which is basically a technology that provides AWS locally.

So this post will be about the rocky road i took to get it running which took me a little longer than 15 minutes.

## Who reads guides anyway!

So I have been wanting to try FaaS (Functions as a Service) which I understand as the next evolution of to micro services. I found some guides and watched some youtube videos showing how easy it was to deploy. I thought how hard could it be!

So I started by flashing the firmware to install Debian Minimal and half way into the process I hit a brick wall due to differences in how the NAT translation in the newer Linux Kernel 5.9 compared to say Ubuntu 20.04 which is still on kernel 4.19. The problem comes from the transition of the normal IPTables to instead use NFTables.

In the hopes of a short solution I tweeted <a href="https://twitter.com/alexellisuk">@alexellisuk</a> who suggested I instead use Ubuntu 20. Alex (the developer of OpenFaaS) recently <a href="https://alexellisuk.medium.com/walk-through-install-kubernetes-to-your-raspberry-pi-in-15-minutes-84a8492dc95a">posted a newer guide</a> suggesting only 15 minutes to get it running using a lightweight kubernetes  (k3s).

## Installing Kubernetes

Now because I was not using a Raspberry Pi, i knew some steps would not apply from the guide that Alex posted. Alex has created some great tools that streamline the process. So long as you run them on the right system. 

Because I did not follow his guide word for word, I missed the part that suggests you don't drive the installation from the target but instead from your Laptop/Desktop instead (which I later gleamed from one of his older videos posted on youtube).

I installed WSL2 running the Ubuntu 20 image on my windows desktop to make it a little easier after my previous failures. You see i tried to putty into the target server and install the tools onto it like that. This did not bode well because I did not know what i was doing and was getting stuck on simple steps like trying to 'ssh-copy-id'.

So the first step is to establish a pre-authenticated session between my workstation and the target server. You need to create an ssh key and this is done on your workstation (not the server).

```
ssh-keygen
```

Don't specify a passphrase which makes future sign-in simple. You can now copy your public ssh key from your workstation to the server.

```
ssh-copy-id -i .ssh/id_rsa.pub root@192.168.1.136
```

Now the above command takes the public ssh key from your local workstation and connects over ssh to the server (192.168.1.136 in my case) and uses the root account to connect. You will provide the password and I can only imagine you could do this with any other account but i am not sure the limitations (like needing sudo ?). Anyhow once that is done you should be able to ssh from your workstation to the server without having to input the password.

ssh root@192.168.1.136

It works by adding your public key into the '~/.ssh/authorized_keys' of the root account (or which ever account you chose) which makes this work.


Now that our workstation is primed to easily connect to the server, we need the tooling that helps remotely deploy k3s to the server.
```
curl -sSL https://dl.get-arkade.dev | sudo sh
```

Alex created a lovely tool called 'arkade' which helps in getting both tools and deploying a selection of pods to the k3s cluster.

```
arkade get k3sup
```

This will download 'k3sup' (pronounced "ketchup") which is a tool to help you remotely install k3s. 

```
export IP=192.168.1.136
```

This exports (sets an environment variable) the target IP address where you will install 'k3s' on.

```
k3sup install --ip $IP --user root
```

This will then install 'k3s' on the target server and you specify the user to log on with using SSH to remotely install on the target server. I used 'root' but you could also use 'pi' if you were installing on a raspberry pi.

During the installation process it will then copy a 'kubeconfig' file over to your workstation in which ever folder you ran the install from. I would suggest that you do this from your home directory.

```
arkade get kubectl
```

This downloads the tool used to interact with 'k3s' to get status or control the cluster as needed. But you will need to make sure you properly configured your environment to point to your kubeconfig file.

```
export KUBECONFIG=/home/jt/kubeconfig
```

In my case it originally saved to my '~/.ssh' folder but you can move it and the above is the environment variable needed to make 'kubectl' connect.

```
kubectl get node -o wide
kubectl top node
kubectl top pod --all-namespaces
```

Some commands to get you started with discovering your freshly installed 'k3s' cluster and checking if the nodes are up and running.

## Your first pod

So the first pod or application you can try install before openfaas, as in the guide, is the 'kubernetes-dashboard'.
```
arkade install kubernetes-dashboard
```

From your workstation this command will deploy the dashboard to your server. Once it's completed you actually need to create an admin user first before you can access the dashboard.

```yaml
cat << EOF | kubectl apply -f -
 apiVersion: v1
 kind: ServiceAccount
 metadata:
   name: admin-user
   namespace: kubernetes-dashboard
 apiVersion: rbac.authorization.k8s.io/v1
 kind: ClusterRoleBinding
 metadata:
   name: admin-user
 roleRef:
   apiGroup: rbac.authorization.k8s.io
   kind: ClusterRole
   name: cluster-admin
 subjects:
 kind: ServiceAccount
 name: admin-user 
   namespace: kubernetes-dashboard
 EOF
```

The above script you just paste direct into your bash window and run which will create the user and role.

```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user-token | awk '{print $1}')
```

This command will print the token created for this admin account that you will need when you try to log into the dashboard.

```
kubectl proxy
```

In order to access the dashboard locally on your workstation hosted on the server, you can proxy it from the server to your local workstation with the above command.

```
http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
```

This is the link to use in order to access the dashboard, it will ask you to input the token. Once you paste the token, you can then browse the dashboard and get information about the cluster.

## Broken Config

So one problem I faced was a bad kubeconfig or something possibly from one of the previous attempts to install 'k3s' both directly on the server and remotely (a couple times). 

I tried draining the old nodes, deleting them, uninstalling k3s using the 'k3s-uninstall' script on the server. But eventually I found a nice nugget.

```
k3sup install --ip $IP --user root --local-path $HOME/.kube/config --merge --skip-install
```

Running this on your workstation will basically recopy the current kubeconfig from your server to your workstation to the 'local-path'. You can also see I changed the location and name of the file.

```
 export KUBECONFIG=/home/jt/.kube/config
```

I had to update my environment to reference the new config file because i didn't like it sitting in my main home directory and this was something I saw Alex do in one of his other videos.

## Reached the goal

Finally some luck and you should be able to finally deploy openfaas to your k3s cluster.

```
arkade install openfaas
```

Hopefully this installs without issue and is able to deploy to your cluster, I kept getting an error regarding the basic-auth not being configured correctly or already setup. I solved this by downloading the kubeconfig again but it could also have been from cleaning the server and all the old configs for 'rancher' that I could find.
```
arkade get faas-cli
```

Running this on your workstation installs the client needed to communicate with the installation of OpenFaaS. You may have noticed some output after installing openfaas that guides you on the next steps. 

```
export PASSWORD=$(kubectl get secret -n openfaas basic-auth -o jsonpath="{.data.basic-auth-password}" | base64 --decode; echo)
export OPENFAAS_URL=http://$IP:31112
```

This sets up the workstation environment with the password and endpoint of the openfaas service. In order to use the 'faas-cli', you need to login first.

```
echo -n $PASSWORD | faas-cli login --username admin --password-stdin
```

This little gem will login and allow you then interact with openfaas using the cli.

```
faas-cli store list --platform armhf
faas-cli store deploy figlet --platform armhf
faas-cli list
```

The following commands on the workstation shows how to list the functions available for deployment in the store. Deploying a simple function 'figlet' to your openfaas instance and finally a list of the functions currently deployed to your openfaas instance.

There is also a web interface you can use to interact with your deployed functions.

```
echo $OPENFAAS_URL
echo $PASSWORD
```

This will display the details on your workstation that you will need to access the dashboard with the username being 'admin'.

The only thing you will need to do now is configure your environment to be more permanent so next time you open your bash they will be set for you.

### Contributing Research Material

[Alex Ellis OpenFaaS in 15 minutes](https://alexellisuk.medium.com/walk-through-install-kubernetes-to-your-raspberry-pi-in-15-minutes-84a8492dc95a)

[Raspberry Pi & Kubernetes with k3s, k3sup, arkade](https://www.youtube.com/watch?v=ZiR3QEfBivk&amp;t=2069s)

[Installing k3s to Ubuntu on Raspberry Pi 4](https://www.youtube.com/watch?v=vzZa-8oXF88&amp;t=451s)

[Kubernetes Homelab with Raspberry Pi 4](https://www.youtube.com/watch?v=qsy1Gwa-J5o&amp;t=494s)

[previous_post]: {% post_url 2020-08-29-my-web-app-journey-data-source-take-2 %}
