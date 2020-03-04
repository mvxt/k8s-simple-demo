# k8s-simple-demo
Simple repository with easy K8s + EKS setup.

## Steps to Run
1. Clone repository
2. Fulfill tool prerequisites:
```bash
# Install or upgrade AWSCLI
$ pip install awscli --upgrade --user

# Configure
$ aws configure
AWS Access Key ID [None]: YOUR_ACCESS_KEY_ID_HERE
AWS Secret Access Key [None]: YOUR_SECRET_ACCESS_KEY_HERE
Default region name [None]: YOUR_REGION_HERE
Default output format [None]: json

# Install kubectl
# Commands vary. See: https://kubernetes.io/docs/tasks/tools/install-kubectl/
```

3. Install `eksctl`. Follow [instructions in AWS Docs](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html#install-eksctl).

4. Create an EKS cluster with the following command (**Note** - This takes a little while, maybe about 30 mins):
```bash
$ eksctl create cluster \
  --name CLUSTER_NAME \
  --version 1.14 \
  --region REGION_HERE \
  --nodegroup-name NODE_GROUP_NAME \ # I used standard-workers
  --node-type TYPE \ # I used t2.medium
  --nodes NUM_NODES \ # I started with 2
  --nodes-min MIN_NODES \ # I started with 1
  --nodes-max 4 \ # I started with 4
  --managed
```

5. The last command should have automatically configured your `kubectl` to point at your AWS EKS cluster. Now, you can create a `deployment` using `kubectl` and the provided `deployment.yml`. Then check that the deployment worked.
```bash
$ kubectl apply -f deployment.yml
deployment.apps/nodejs created
$ kubectl get deployments.
NAME     READY   UP-TO-DATE   AVAILABLE   AGE
nodejs   1/1     1            1           7s
```

6. Then, after the deployment is done, it's time to create the service with `kubectl` and the provided `service.yml`. After you run the command, it will take some time before the service is actually ready (I waited about 15 mins?). You can confirm by running follow-up commands to check the status of everything:
```bash
# Should say configured
$ kubectl apply -f service.yml
service/nodejs configured

# Should say successfully rolled out
$ kubectl rollout status deployment nodejs
deployment "nodejs" successfully rolled out

# You should see something under EXTERNAL-IP pointing to a load balancer.
$ kubectl get services --all-namespaces -o wide
NAMESPACE     NAME         TYPE           CLUSTER-IP       EXTERNAL-IP                                                               PORT(S)         AGE   SELECTOR
default       kubernetes   ClusterIP      123.123.123.123  <none>                                                                    443/TCP         25m   <none>
default       nodejs       LoadBalancer   123.123.123.123  blah-blah-blah-blah-blah-blah-blah-blah-etc.us-east-1.elb.amazonaws.com   80:31926/TCP    98s   app=nodejs

# Should say Running unders STATUS
$ kubectl get pods -o wide --watch
NAME                      READY   STATUS    RESTARTS   AGE     IP             NODE                             NOMINATED NODE   READINESS GATES
nodejs-84cdb9f66f-hz524   1/1     Running   0          4m30s   123.123.123.12 ip-123-123-123-123.ec2.internal  <none>           <none>

# Rerun the "get deployments" command, and you should see the output is different compared to step 5.
# There should be new CONTAINERS, IMAGES, and SELECTOR attached to the deployment now.
$ kubectl get deployments -o wide
NAME     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS        IMAGES                            SELECTOR
nodejs   1/1     1            1           8m45s   nodejs-k8s-demo   mikeyvxt/nodejs-k8s-demo:latest   app=nodejs
```

7. If you've confirmed output above, your app should be reachable via the EXTERNAL-IP hostname given in the `get services` command above. **NOTE:** The app is configured for HTTP / Port 80 access in browser, so if you're getting an error, make sure your URL says `http://` and not `https://`.

If you want to use your own Docker image or configure your own app, you'll need to modify the `deployment.yml` and `service.yml` files.

