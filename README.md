Kubernetes Installation on Rocky Linux
This repository contains the necessary steps and scripts to install and configure Kubernetes on Rocky Linux. The setup includes a single-node or multi-node cluster using kubeadm.

Prerequisites
Before proceeding, ensure the following:

Rocky Linux 8.x or 9.x installed on all nodes.

At least 2 GB of RAM and 2 CPUs per machine.

Unique hostnames for each node.

SSH access to all nodes.

Root or sudo privileges on all nodes.

Firewall and SELinux configured (or disabled for simplicity).

Setup Overview
Master Node: Runs the control plane components (API Server, Scheduler, Controller Manager, etc.).

Worker Nodes: Runs the workloads (pods and containers).

Networking: Uses a CNI (Container Network Interface) plugin like Calico or Flannel.

Installation Steps
1. Update System and Install Dependencies
Run the following commands on all nodes (master and workers):

bash
Copy
sudo dnf update -y
sudo dnf install -y curl vim git
2. Disable Swap
Kubernetes requires swap to be disabled:

bash
Copy
sudo swapoff -a
sudo sed -i '/swap/d' /etc/fstab
3. Install Docker or Containerd
Kubernetes requires a container runtime. You can use Docker or Containerd.

Option 1: Install Docker
bash
Copy
sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install -y docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
Option 2: Install Containerd
bash
Copy
sudo dnf install -y containerd
sudo systemctl enable --now containerd
4. Configure Containerd (if using Containerd)
Edit the Containerd configuration file:

bash
Copy
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
5. Install Kubernetes Components
Add the Kubernetes repository and install kubeadm, kubelet, and kubectl:

bash
Copy
cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kube*
EOF

sudo dnf install -y kubelet kubeadm kubectl --disableexcludes=kubernetes
sudo systemctl enable --now kubelet
6. Initialize the Kubernetes Cluster (Master Node Only)
On the master node, run:

bash
Copy
sudo kubeadm init --pod-network-cidr=10.244.0.0/16
After initialization, you will see a command to join worker nodes. Save this command for later.

Set up kubeconfig for the current user:

bash
Copy
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
7. Install a CNI Plugin (Master Node Only)
Install a CNI plugin like Calico:

bash
Copy
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
8. Join Worker Nodes
On each worker node, run the kubeadm join command provided during the master node initialization.

Example:

bash
Copy
sudo kubeadm join <master-ip>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash>
9. Verify the Cluster
On the master node, check the status of the nodes:

bash
Copy
kubectl get nodes
You should see all nodes in the Ready state.

Additional Configuration
Enable Shell Autocompletion for kubectl
To enable autocompletion for kubectl, run:

bash
Copy
echo 'source <(kubectl completion bash)' >> ~/.bashrc
source ~/.bashrc
Deploy a Sample Application
Deploy a sample Nginx application to test the cluster:

bash
Copy
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc nginx
Troubleshooting
If nodes are not joining the cluster, ensure that the firewall allows traffic on port 6443.

Check the logs using journalctl -u kubelet for any errors.

Ensure that the container runtime (Docker/Containerd) is running.

References
Kubernetes Official Documentation

Calico Networking

Rocky Linux Documentation

License
This project is licensed under the MIT License. See the LICENSE file for details.

Feel free to contribute or open issues for any problems you encounter!
