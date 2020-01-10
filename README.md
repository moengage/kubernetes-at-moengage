<img src="https://github.com/kubernetes/kubernetes/raw/master/logo/logo.png" width="100px" height="100px"><img src="https://github.com/moengage/kubernetes-at-moengage/raw/master/static-assets/moengage.png" width="200px" height="100px">

# kubernetes@moengage

This repo consists of setup processes of opensource tools, adoption of them in moengage, tools that
we extended and few tools that we built to run our services at scale on production.

Provisioning
------------

Most of the components have README included.

- Create EKS Cluster using [tf-core/eks_cluster](https://github.com/moengage/tf-core/tree/master/eks_cluster)
- Make networking changes, this is a manual and necessary configuration to handle pod networking on secondary VPC CIDR, so make sure you don't run any nodes else services may behave abnormally. Have a look [here](https://github.com/moengage/kubernetes-at-moengage/tree/master/00networking)

