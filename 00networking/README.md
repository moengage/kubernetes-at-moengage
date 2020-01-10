When we create kubernetes clusters, we make following changes initially

Custom pod networking with secondary VPC CIDR
---------------------------------------------

1. Pre-allocation of IP addresses has been reduced to a lower number, check `WARM_IP_TARGET` & `WARM_ENI_TARGET`:
    ```
    kubectl set env daemonset aws-node -n kube-system AWS_VPC_K8S_CNI_CUSTOM_NETWORK_CFG=true WARM_IP_TARGET=4 AWS_VPC_K8S_CNI_EXTERNALSNAT=true WARM_ENI_TARGET=10
    ```

2. Change ENI config so that pods get IPs from secondary
   IP address pool of the VPC. We need to follow following to tweak kubernetes
   ENI config little bit.
   - Assign one or more CIDR address as secondary CIDR
   - Create a subnet or multiple subnets with this range in current VPC and
     other dependent resources like routetables/nacls etc. It is recommended to
     use terraform here, so that manual overhead is less.
   - Make sure to tag the subnet with: `kubernetes.io/cluster/<cluster-name>: owned`
   - Make a note of worker security group ID that got created when you ran
     terraform cluster creation using [tf-core/eks_cluster](https://github.com/moengage/tf-core/tree/master/eks_cluster)
   - Apply CRD: `./eni-config-crd.yaml`
   - Create manifests using `./eni-config-template.yaml` for each subnet you created
     above. Worker security group is nodegroup security groups.
   - I'll assume we are doing all these in a new cluster, so any node joining
     should use the above ENI Config, for that we need to mention the same in
     bootstrap args. Make sure the node group is getting launched in one subnet
     only and contains the following in userdata: `"--kubelet-extra-args '--node-labels=k8s.amazonaws.com/eniConfig=<eni-config-name>'"`
     So, I would suggest to use [tf-core/eks_nodegroups](https://github.com/moengage/tf-core/tree/master/eks_nodegroup) to create worker nodes
     and pass bootstrap_extra_args = the above line and you are done.
   - Note: We do not use automatic AZ based ENI config selection.


Change kube-proxy-config to support prometheus scraping
-------------------------------------------------------

Default kube-proxy installation doesn't expose metrics port, to enable prometheus scraping, we are
making metrics port available on host port: `./kube-proxy-cm.yaml`


References
----------
  - https://docs.aws.amazon.com/eks/latest/userguide/cni-custom-network.html
  - https://docs.aws.amazon.com/en_ca/eks/latest/userguide/cni-env-vars.html
