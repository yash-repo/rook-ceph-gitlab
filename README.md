

## Gitlab Deployments On Kubernetes Cluster

Manual Steps:-

REQUIREMENTS

1. A domain to which you or your company owns, to which you can add a DNS record. 
    Make a subdomain of the parent domain and add the nameservers of the subdomain to    parent domain (eg. parent.com & dev.parent.com)

2. A Kubernetes cluster. 
3. A working installation of kubectl
4. A working installation of Helm v3.

Steps Involved:-

1. Adding the GitLab Helm repository
    helm repo add gitlab https://charts.gitlab.io/

2. Installing GitLab
    helm install gitlab gitlab/gitlab --set global.hosts.domain=gitlab.gitlab.labcerebrone.com --set certmanager-issuer.email=me@gitlab.gitlab.labcerevrone.com --set gitlab-runner.install=false

  Here, we have set the attributes of our helm chart such as providing the domain name, etc . For gitlab-runner we have set it to false as the runner requires certain parameters to run such as certificates , runners won’t run on self-signed certificates.

3. Retrieve the IP address
   We can use kubectl to fetch the address that has been dynamically allocated by our Kubernetes to the NGINX Ingress we’ve just installed and configured as a part of the GitLab chart. 
   kubectl get ingress -lrelease=gitlab

We’ll notice there are 3 entries, and they all have the same IP address. We’ll need to take this IP address, and add it to your DNS for the domain you have chosen to use. You can add 3 separate records of type A, but we suggest adding a single “wildcard” record for simplicity. In Route53, this is done by creating an A record/CNAME can also work, but with the name being *. We also suggest you set the TTL to 1 minute instead of 5 minutes



4. Sign in to GitLab
   In order to sign in, we’ll need to collect the password for the root user. This is automatically generated at installation time, and stored in a Kubernetes Secret. Let’s fetch that password from the secret, and decode it:
kubectl get secret gitlab-gitlab-initial-root-password -ojsonpath='{.data.password}' | base64 --decode ; echo

5. For getting the Runner Token use:-
   kubectl get secret gitlab-gitlab-runner-secret  -ojsonpath='{.data.runner-registration-token}' | base64 --decode ; echo

6. Clone the Runner Chart in Your Local Machine
    git clone https://gitlab.com/gitlab-org/gitlab-runner.git

7. Edit the Values.yaml file in the above chart and add these to it:-
    gitlabUrl: https://gitlab.gitlab.labcerebrone.com
  runnerRegistrationToken: "<runner-token"
  privileged: "true"

8. Apply Changes and Install the Chart
   helm install -f values.yaml gitlab-runner <path-of-directory>


Make Storage Class default

• kubectl patch storageclass rook-ceph-block -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'


Adding Certificates For Gitlab-UI

so we have added a certificate from acm (gitlab.gitlab.labcerebrone.com)and added the name,value,cname to route53 to give gitlab access to the web using certificates
steps to add certificate to loadbalancer of webservice
1. find the port gitlab-webservice is running i.e 32075
go to ec2 console and head to loadbalancer section,
over there click on the loadbalncer and head to listeners and check the port with 32075
edit that as:-
https 443 https 32075 and provide the certificate(choose acm).
