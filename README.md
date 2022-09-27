# Deploy EKS with EKS Blue Print


## Step 1 - Deploying Production Ready EKS 

EKS Blueprints allows you to configure and deploy what we call blueprint clusters. A blueprint combines clusters, add-ons, and teams into a cohesive object that can be deployed as a whole. Once a blueprint is configured, it can be easily deployed across any number of AWS accounts and regions. Blueprints also leverage GitOps tooling to facilitate cluster bootstrapping and workload onboarding.

![EKS Blue Print](https://static.us-east-1.prod.workshops.aws/public/ef3abd77-2c05-44ed-a9fc-471c2e36dfab/static/images/blueprint.png)

EKS Blueprints provide a better workflow between infrastructure and development teams, and also provide a self-service interface for developers to use that is streamlined for developing code. The infrastructure teams have full control to define standards on security, software delivery, monitoring, and networking that must be used across all applications deployed. This allows developers to be more productive because they don’t have to configure and manage the underlying cloud resources themselves. It also gives operators more control in making sure production applications are secure, compliant, and highly available.

1. Install CDK
`npm install -g aws-cdk@2.37.1`

2. Do npm install

`npm install`

3. Bootstrap your account with CDK

```shell
cdk bootstrap --trust=$ACCOUNT_ID \
  --cloudformation-execution-policies arn:aws:iam::aws:policy/AdministratorAccess \
  aws://$ACCOUNT_ID/$AWS_REGION aws://$ACCOUNT_ID/us-east-2 aws://$ACCOUNT_ID/us-east-1
  ```
4. Deploy 

`
cdk deploy
`

## Step 2 - Deploy App with Nginx Ingress

In Kubernetes, these are several different ways to expose your application; using Ingress to expose your service is one way of doing it. Ingress is not a service type, but it acts as the entry point for your cluster. It lets you consolidate your routing rules into a single resource, as it can expose multiple services under the same IP address.

THis Github repo will explain how to use an ingress resource and front it with a NLB (Network Load Balancer). 

![Deployment Architecture](https://d2908q01vomqb2.cloudfront.net/fe2ef495a1152561572949784c16bf23abb28057/2020/08/01/NLB-NGINX-Architecture.png)


The below manifest file also launches the Network Load Balancer(NLB).

`kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/aws/deploy.yaml`

Now create two services (apple.yaml and banana.yaml) to demonstrate how the Ingress routes our request.  We’ll run two web applications that each output a slightly different response. Each of the files below has a service definition and a pod definition.

```shell
kubectl apply -f https://raw.githubusercontent.com/cornellanthony/nlb-nginxIngress-eks/master/apple.yaml 
kubectl apply -f https://raw.githubusercontent.com/cornellanthony/nlb-nginxIngress-eks/master/banana.yaml
```

Now declare an Ingress to route requests to /apple to the first service, and requests to /banana to second service. Check out the Ingress’ rules field that declares how requests are passed along:


```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "false"
spec:
  rules:
    - http:
        paths:
          - path: /apple
            backend:
              serviceName: apple-service
              servicePort: 5678
          - path: /banana
            backend:
              serviceName: banana-service
              servicePort: 5678
```