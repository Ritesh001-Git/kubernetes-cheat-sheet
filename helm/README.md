# Helm
If Kubernetes resources (Deployments, Services, Ingress, ConfigMaps) are the individual ingredients, Helm is the Recipe Book and the Chef.

In the tech industry, nobody manually runs kubectl apply -f on 50 different YAML files to deploy an application. Instead, we use Helm, which is the official package manager for Kubernetes (essentially the apt, yum, or npm of the cloud-native world).

1. What is Helm?
Helm allows you to package all of your Kubernetes YAML files into a single, reusable folder called a Chart. It then templates those files so you can dynamically change values (like image versions, replicas, or passwords) depending on whether you are deploying to Dev, Staging, or Production.

Core Concepts:
Chart: A blueprint bundle of Kubernetes resources (a package).

Values: A configuration file (values.yaml) containing the variables that customize the template.

Release: A specific instance of a Chart running inside your cluster. For example, you can deploy the "WordPress" chart twice, creating a blog-release and a shop-release.
