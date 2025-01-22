# Meta LLama Model Inference on Inferentia2 using EKS
This is a sample project that uses Elastic Kubernetes Service (EKS) and Deep Java Library (open-source) to host transformer models from huggingface.co
on Inferentia2 (Amazon Accelerated Compute Instance)

## Features

- Read Deep Java Library(DJL) here: [DJL](https://docs.djl.ai/master/)
- Purpose built compute ([inferentia2](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/general/arch/neuron-hardware/inf2-arch.html#aws-inf2-arch)) for LLM Inference
- Dive into [Inferentia chip architecture](https://awsdocs-neuron.readthedocs-hosted.com/en/latest/general/arch/neuron-hardware/inferentia2.html#inferentia2-arch)
- Cost Effective

## Prerequisites

List what needs to be installed before running your project:

- eksctl & kubectl cli.
- Need Amazon Web Services Account (dev/admin) priviledges
- Service Quota in AWS account/ region (default:us-west-2) to spin up inf2.xlarge instance
- Optional huggingface auth token and subscription to Meta's Llama model

## Installation

Provide step-by-step installation instructions:

```bash
# Installation commands
git clone https://github.com/arnabaws15/eks-djl-neuronx.git
cd eks-djl-neuronx
source env_vars # update env_vars file if you want to change defaults

# create EKS cluster
eksctl create cluster --config-file eks_cluster.yaml  # create eks cluster

# Add inf2 managed node
eksctl create nodegroup \
    --config-file eks_nodegroup.yaml \
    --install-neuron-plugin=false

# Manually enable neuron device plugin for EKS
kubectl apply -f https://raw.githubusercontent.com/aws-neuron/aws-neuron-sdk/master/src/k8/k8s-neuron-device-plugin-rbac.yml
kubectl apply -f https://raw.githubusercontent.com/aws-neuron/aws-neuron-sdk/master/src/k8/k8s-neuron-device-plugin.yml

# Schedule the Neuron Schedular Extension
kubectl apply -f https://raw.githubusercontent.com/aws-neuron/aws-neuron-sdk/master/src/k8/k8s-neuron-scheduler-eks.yml
kubectl apply -f https://raw.githubusercontent.com/aws-neuron/aws-neuron-sdk/master/src/k8/my-scheduler.yml

# Verify plugin installation
kubectl get ds neuron-device-plugin-daemonset --namespace kube-system
# Verify node can allocate neuron cores/devices
kubectl get nodes "-o=custom-columns=NAME:.metadata.name,NeuronCore:.status.allocatable.aws\.amazon\.com/neuroncore" # should be 2 for inf2.xlarge

# Deploy llama2 pod
kubectl apply -f  deploy_djl.yaml 

# Verify pods are up
kubectl get po

# Check output from pod
kubectl logs pod/llama2-pod # look for API BIND to http://0.0.0.0:8080

# Query for inference
kubectl get service/llama2-service # grab the host and port for curl command below
curl -X POST <EXTERNAL-IP>:<PORT>/invocations   -H "Content-type: application/json" \
   -d '{"inputs":"tell me a story of the little red riding hood","parameters":{"max_new_tokens":256, "do_sample":true}}' 
```

## License
This sample is lincesed under the Apache-2.0 License

