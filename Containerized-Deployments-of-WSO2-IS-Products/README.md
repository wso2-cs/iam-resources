# **Containerized Deployments of WSO2 Products**

## **Best Practices and Guidelines**



# Table of Contents

[Table of Contents](#table-of-contents)

[1.0 Recommendations & best practices](#10-recommendations--best-practices)

* [1.1 Building Docker Images](#11-building-docker-images)

* [1.2 Image Registry](#12-image-registry)

* [1.3 Kubernetes](#13-kubernetes)

[2.0 Development recommendations and best practices](#20-development-recommendations-and-best-practices)

* [2.1 Importance of using Docker](#21-importance-of-using-docker)

* [2.2 Importance of using official WSO2 product Docker images](#22-importance-of-using-official-wso2-product-docker-images)

* [2.3 Extending official Docker images for WSO2 products](#23-extending-official-docker-images-for-wso2-products)

* [2.4 Build a WSO2 Product Docker Image from scratch](#24-build-a-wso2-product-docker-image-from-scratch)

[3.0 Deployment recommendations and best practices for containerized environments](#30-deployment-recommendations-and-best-practices-for-containerized-environments)

* [3.1 Always use a private container image registry to pull images for a container-based deployment](#31-always-use-a-private-container-image-registry-to-pull-images-for-a-container-based-deployment)

* [3.2 Always use a container orchestration platform to deploy containers in production](#32-always-use-a-container-orchestration-platform-to-deploy-containers-in-production)

* [3.3 Kubernetes is the best container orchestration platform for WSO2 products](#33-kubernetes-is-the-best-container-orchestration-platform-for-wso2-products)

[4.0 WSO2 Kubernetes deployment resources and best practices](#40-wso2-kubernetes-deployment-resources-and-best-practices)

* [4.1 Keeping a Pod healthy with Kubernetes](#41-keeping-a-pod-healthy-with-kubernetes)

* [4.2 Resource limits configurations](#42-resource-limits-configurations)

* [4.3 Configure Horizontal Pod Autoscaling (HPA) based on CPU](#43-configure-horizontal-pod-autoscaling-hpa-based-on-cpu)

* [4.4 Monitoring WSO2 resources](#44-monitoring-wso2-resources)

* [4.5 Maintaining multiple environments in a K8S deployment](#45-maintaining-multiple-environments-in-a-k8s-deployment)

* [4.6 WSO2 product deployed with WSO2 Helm resources](#46-wso2-product-deployed-with-wso2-helm-resources)

* [4.7 Avoid creating naked pods with WSO2 products](#47-avoid-creating-naked-pods-with-wso2-products)

* [4.8 Using persistence storage with WSO2 products](#48-using-persistence-storage-with-wso2-products)

* [4.9 Internal services exposed with an Ingress resource](#49-internal-services-exposed-with-an-ingress-resource)

* [4.10 Updating an existing WSO2 product deployment](#410-updating-an-existing-wso2-product-deployment)

	* [4.10.1 U2 update on K8s : How to do it?](#4101-u2-update-on-k8s--how-to-do-it)

	* [4.11 Enable alerts in order to identify deployment failures](#411-enable-alerts-in-order-to-identify-deployment-failures)

[5.0 Further Reading](#50-further-reading)

#


# 1.0 Recommendations & best practices

This section outlines the development, deployment and Kubernetes best practices for WSO2 products.

## 1.1 Building Docker Images

- Use official WSO2 Docker container images in evaluatory, containerized deployments of [WSO2 products](https://docker.wso2.com/tags.php?repo=wso2is).
- Use an official WSO2 Docker container image as the base, when building your custom, WSO2 product Docker image (e.g. in an event where additional binaries are to be packaged).
- Use an official GitHub release (e.g. release tag version [v5.11.0.15](https://github.com/wso2/docker-is/tree/v5.11.0.15) of Docker resources for WSO2 IS) of the Docker resources of the relevant WSO2 product, when building your custom WSO2 product Docker image from source (e.g. in an event where, new software packages are to be installed).
- Build required binaries such as keystores and deployment artifacts into the WSO2 product Docker container image.
- These images need to be then pushed to a private container image registry. See section [1.2](#12-image-registry) for details.

For more details, please refer to section [2.0](#20-development-recommendations-and-best-practices).

## 1.2 Image Registry

A user must use a private container image registry of his/her own, to host images and pull from, in a container-based deployment.

This approach is recommended due to the following reasons.

- It is easy to keep track of the contents of the container images used within the user's deployment environments.
- The container image may package contents, which should be accessible only to the specific user (e.g. customer specific artifacts, [WSO2 Updates](https://wso2.com/updates/) obtained via the user's [WSO2 Subscription](https://wso2.com/subscription)).
- Rolling back after the update to the previous state, WSO2 only keeps the latest few images in the WSO2 private repository.

For more details, please refer to section [3.1](#31-always-use-a-private-container-image-registry-to-pull-images-for-a-container-based-deployment).

## 1.3 Kubernetes

WSO2 recommends Kubernetes as the container orchestration platform. The following best practices should be followed, when using Kubernetes in a production grade deployment.

- Use Helm charts released by WSO2 for product deployment.
- Use Docker image digest for pulling images.
- Use [ConfigMaps](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/) to induce changes to WSO2 product configuration files.
- Use [Liveness and Readiness probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) to keep the pod healthy in the Kubernetes cluster.
- Define [Resource limit](https://kubernetes.io/docs/concepts/policy/resource-quotas/#compute-resource-quota) configurations for the product deployment.
- Configure [Horizontal Pod Autoscaling (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) based on CPU.
- Use an [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) to expose an internal service.
- Connect the WSO2 Product resources with an external, centralized log management system such as [ELK](https://www.elastic.co/what-is/elk-stack) (Elasticsearch, Logstash and Kibana or Stackdriver can be used for GKE)
- Monitor WSO2 products using metrics monitoring tools such as [Prometheus with Grafana](https://prometheus.io/docs/visualization/grafana/), [Appdynamics](https://www.appdynamics.com/), and [Dynatrace](https://www.dynatrace.com/). Areas that need to be covered are as follows:
  - JMX monitoring (Refer this [document](https://is.docs.wso2.com/en/latest/setup/jmx-based-monitoring/))
  - JVM monitoring
  - Probe monitoring
  - WSO2 products resource usage
- [Enable alerts](https://prometheus.io/docs/alerting/alertmanager/) in order to identify the deployment failures.
- Isolate low level environments such as development and staging by using Kubernetes [namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). This reduces administrative overhead, maintenance, and cost due to improved utilization.
- The higher level environments such as production environments should be isolated using independent Kubernetes clusters. This isolation would help for load testing, security and VPN configurations and node auto-scaling.

For more details, please refer to section [4.0](#40-wso2-kubernetes-deployment-resources-and-best-practices).

#


# 2.0 Development recommendations and best practices

This section covers (in detail) the WSO2-recommended best practices and guidelines for preparing Docker container images.

## 2.1 Importance of using Docker

This section explains the reasons for why WSO2 has chosen to provide container images for the Docker container platform.

WSO2 provides container images, which are required for deploying its products/solutions on the Docker container platform. This is due to the following reasons:

- High popularity and customer demand.
- High usability and advanced features.
- Wide community support towards the Docker container platform.

## 2.2 Importance of using official WSO2 product Docker images

WSO2-curated product Docker images are available for usage from [Docker Hub](https://hub.docker.com/u/wso2/) and [WSO2's Private Docker Registry](https://docker.wso2.com/).

WSO2 recommends using its official Docker container images in containerized deployments of WSO2 products.

This is because,

- WSO2 has aligned its Docker images with its recommended best practices for product deployments.

Following are some of the features of WSO2 product Docker images, which aligns them with the WSO2-recommended best practices for product deployments:

- Supports the application of changes to the default configurations of a WSO2 product (refer [this](https://medium.com/faun/wso2-products-dockerized-advanced-usage-part-1-ae255ef02661) article for advanced details).
- Supports the introduction of artifacts/non-configuration files (e.g. third-party libraries, Carbon extensions in the form of OSGi bundles, [Carbon Applications](https://docs.wso2.com/display/Carbon440/Working+with+Carbon+Applications), or any security-related artifacts such as Java Keystore files) to a deployment (refer [this](https://medium.com/faun/wso2-products-dockerized-advanced-usage-part-2-9c851b0db557) article for advanced details).
- Supports sharing/persisting runtime artifacts in relevant WSO2 product profiles.
- Supports passing WSO2 product startup options (refer [this](https://medium.com/faun/wso2-products-dockerized-remote-debug-dockerized-wso2-products-with-intellij-idea-111ca8ae60c) article for advanced details).

- WSO2 has adopted the best practices associated with implementing Dockerfiles and associated resources.

WSO2 has followed an array of [best practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/) and [guidelines](https://github.com/docker-library/official-images#review-guidelines) recommended by the Docker community during the implementation of its product Docker resources.

Some of the best practices and guidelines that are followed for Dockerfiles and entry point scripts are explained below.

- Minimize the Docker container image size.
  - Reduced number of instructions that would create sizeable image layers, leading to minimize the image size. Refer additional details( link [section](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#minimize-the-number-of-layers)).
  - The product distribution pack and its dependencies are removed to prevent the Docker image size from increasing. Only the necessary software packages have been installed (refer [section](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#dont-install-unnecessary-packages) for advanced details).
- Set the default Docker image user to a non-root user. WSO2-released Docker images ship with the 'wso2carbon' non-root user for running WSO2 products.
- Heightened Clarity.
  - Sorted multi-line arguments (refer [section](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#sort-multi-line-arguments) for advanced details)
  - For additional details on the best practices and guidelines, please see [[1](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)] and [[2](https://github.com/docker-library/official-images#review-guidelines)].
- The usage of a well-defined and curated Docker container image tagging mechanism.

Please refer to [this](https://medium.com/faun/wso2-products-dockerized-image-tagging-mechanism-9b4499831548) comprehensive guide on WSO2's product Docker container image tagging mechanism.

## 2.3 Extending official Docker images for WSO2 products

WSO2 recommends using an official WSO2 product Docker image as the base of your custom Docker image. For further details on this recommendation, refer to the above section [2.2](#22-importance-of-using-official-wso2-product-docker-images).

## 2.4 Build a WSO2 Product Docker Image from scratch

Use the official GitHub release (e.g. release tag version [v5.11.0.15](https://github.com/wso2/docker-is/tree/v5.11.0.15) of Docker resources for WSO2 Identity Server) of the Docker resources of the relevant WSO2 product to build a WSO2 product Docker image by source. For advanced details on recommending official WSO2 product Docker resources for this purpose, you may refer to the above section [2.2](#22-importance-of-using-official-wso2-product-docker-images).

# 3.0 Deployment recommendations and best practices for containerized environments

## 3.1 Always use a private container image registry to pull images for a container-based deployment

This is essential because most public registries do not preserve all images they publish for a long time.

This is applicable to WSO2 product Docker images available at [Docker Hub](https://hub.docker.com/u/wso2/) and [WSO2 Private Docker Registry](https://docker.wso2.com/), as well. A product Docker image may change due to the following reasons.

- Change(s) to the source used to build the Docker image
- Addition of new, WSO2 Updates to the packaged, WSO2 product (applicable _only_ to images available at WSO2 Private Docker Registry)

Once a change occurs, the images are usually replaced with the _latest_ having the same tag.

Hence, in a case where a customer's deployment is updated with a newly pulled image (without the knowledge of the exact change) and it leads to defect(s) in the deployment, it can be difficult to identify the exact reason for each defect. Further, there is no way of rolling back after the update to the previous state, as history is not retained.

WSO2 recommends the use of a user's own private container image registry, to keep track of all the used images, so that the recovery of a previous version is easy at any time with zero risk and zero dependence on outside sources.

## 3.2 Always use a container orchestration platform to deploy containers in production

Even though it is easy to manage, maintain, and run all your containers in a single node cluster, this practice could lead to a single point of failure (if the node suddenly malfunctions due to some problems). As a result, it's always advisable to divide your containers into two or more nodes in a cluster. However, depending on the load, we not only have to increase the number of containers we manage, but we will also have to increase the number of nodes we manage for the cluster. Nevertheless, the higher the number of containers and nodes, the higher the effort and complexity of managing the cluster. Doing this manually is a daunting task and is highly error prone.

This is where a container orchestration platform comes into the picture as a solution to the problem. It can manage all of the bare metal machines or virtual machines on which you need to run your containers. Furthermore, it could also manage your containers by launching them on the underlying machines, making sure they are distributed, and keeping them healthy.

Hence, we recommend Kubernetes as the platform for container orchestration.

## 3.3 Kubernetes is the best container orchestration platform for WSO2 products

When it comes to the three leading container orchestration platforms, i.e., Mesos, Kubernetes, and Docker Swarm, Kubernetes is the undisputed leader due to the following reasons:

- Wide adoption
- Proven, battle-tested solution
- Open source
- High Availability
- Scalability
- Portability
- Powerful and easy-to-use features

![](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/bltc90ec62b675e594e/5bd0b07f5607f44e72b5b111/Kubernetes-graph.png)

Source: [https://www.weave.works/technologies/the-journey-to-kubernetes](https://www.weave.works/technologies/the-journey-to-kubernetes/)

With its wide adoption and powerful, easy-to-use features, we at WSO2 believe that it can soon become the de facto container orchestration platform standard in the world. It is for this reason that we continue to provide development support for Kubernetes-based container deployments of our product (as we have done for more than two years).

# 4.0 WSO2 Kubernetes deployment resources and best practices

## 4.1 Keeping a Pod healthy with Kubernetes

- **Liveness probe** - This detects the containers that are not able to recover from a failed state and need to be restarted. It is recommended to use the **HTTP GET** probe mechanism with WSO2 Products.
- **Readiness probe** - Assess whether the container is ready to receive the traffic. Therefore, it is recommended to use the HTTP GET probe in the deployment specification. This ensures that the HTTP GET request is sent to the container. Further, the HTTP status code of the response determines whether the container is ready.

Examine liveness probe configuration specification given below:
```
livenessProbe:
    exec:
      command:
        - /bin/sh
        - -c
        - nc -z localhost 9443
    initialDelaySeconds: 60            
    periodSeconds: 10

readinessProbe:
    exec:
      command:
          - /bin/sh
          - -c
          - nc -z localhost 9443
    initialDelaySeconds: 60
    periodSeconds: 10
```
## 4.2 Resource limits configurations

WSO2 recommends the use of following CPU and memory 'requests' and 'limits' resource values in a deployment specification. The following memory limits configurations ensure that avoid causing Java OutOfMemory errors for Java processes.
```
resources:
  requests:
    memory: "2Gi"
    cpu: "2000m"

  limits:
    memory: "4Gi"
    cpu: "2000m"
```
## 4.3 Configure Horizontal Pod Autoscaling (HPA) based on CPU

This guarantees that the number of running WSO2 Kubernetes Pods in the deployment scales. Autoscaling should be configured with the average CPU threshold between 80%-90%.

## 4.4 Monitoring WSO2 resources

When monitoring WSO2 products, the areas listed below need to be covered using proper metrics monitoring tools and visualization tools. For example [Prometheus with Grafana](https://prometheus.io/docs/visualization/grafana/), [Appdynamics](https://www.appdynamics.com/), or [Dynatrace](https://www.dynatrace.com/).

1. JMX monitoring
2. JVM monitoring

3. Probe monitoring

4. WSO2 products resource usage

## 4.5 Maintaining multiple environments in a K8S deployment

Low-level environments, such as development and staging, should be isolated using Kubernetes [Namespaces](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/). This reduces maintenance, administrative overheads plus costs and improves utilization. High level environments should be isolated with different Kubernetes clusters, due to new version upgrades, load tests, isolation for security and VPN configuration and auto scaling.

## 4.6 WSO2 product deployed with WSO2 Helm resources

[Helm](https://v2.helm.sh/) is a package manager for Kubernetes, which can be used to deploy and manage application packages in a Kubernetes cluster.

Please see the WSO2 product Helm Charts available at Helm Hub from [here](https://github.com/wso2/kubernetes-is/blob/master/advanced/is-pattern-1/README.md).

## 4.7 Avoid creating naked pods with WSO2 products

Avoid creating [Naked Pods](https://kubernetes.io/docs/concepts/configuration/overview/#naked-pods-vs-replicasets-deployments-and-jobs) because they will not be rescheduled in the event of a node failure.

Kubernetes [Deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) are used to create Kubernetes Pods of WSO2 product profiles in WSO2 product Helm Charts.

## 4.8 Using persistence storage with WSO2 products

WSO2 product deployments may require a persistent storage solution for persisting and/or sharing artifacts created during the runtime of certain server profiles (e.g. WSO2 Identity Server needs to share volumes between nodes).

We recommend the usage of Kubernetes [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/) with ReadWriteMany access mode (please see the Kubernetes official [documentation](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) for appropriate solutions).

WSO2 product Helm Charts uses the Kubernetes [NFS Server Provisioner](https://github.com/helm/charts/tree/master/stable/nfs-server-provisioner), for this purpose.

## 4.9 Internal services exposed with an Ingress resource

Kubernetes [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/) resources are capable of determining traffic based on the host and path forwarded to them by backends. It also operates on the application layer of the network stack (HTTP) and can provide features like cookie-based session affinity and SSL terminations.

## 4.10 Updating an existing WSO2 product deployment

WSO2 recommends the use of Kubernetes [Rolling Updates](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment) to perform an update to an existing WSO2 product deployment in a Kubernetes environment. In order to roll back to a previous revision of the deployment, it is recommended to use Kubernetes [Rollouts](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-back-a-deployment).

### 4.10.1 U2 update on K8s : How to do it?

- Check the U2 diff between the existing U2 level and the updating U2 level and manually merge any update coming to the templated files.
- Pull and tag the latest U2 docker image to the private registry.
- docker pull docker.wso2.com/wso2is:5.11.0.\<U2 level\>
- Use the tagged U2 image in the Dockerfile when building the custom docker image
- Deploy

The WSO2 recommends updating the lower environment bi-weekly and production environment on a monthly basis. However, it is up to the customer to come up with a proper update-schedule depending on the testing capacity and other factors.

## 4.11 Enable alerts in order to identify deployment failures

Prometheus [Alertmanager](https://prometheus.io/docs/alerting/alertmanager/) can be configured with scraped Prometheus agent metrics. Therefore, define the rules based on metrics that can trigger the notification or pager when the deployment failure occurs.

#


# 5.0 Further Reading

1. WSO2 Products Dockerized: An Introduction[https://medium.com/@chirangaalwis/wso2-products-dockerized-an-introduction-5045a197befa](https://medium.com/@chirangaalwis/wso2-products-dockerized-an-introduction-5045a197befa)
2. WSO2 Products Dockerized: Advanced Usage — Part 1 [https://medium.com/faun/wso2-products-dockerized-advanced-usage-part-1-ae255ef02661](https://medium.com/faun/wso2-products-dockerized-advanced-usage-part-1-ae255ef02661)
3. WSO2 Products Dockerized: Advanced Usage — Part 2 [https://medium.com/faun/wso2-products-dockerized-advanced-usage-part-2-9c851b0db557](https://medium.com/faun/wso2-products-dockerized-advanced-usage-part-2-9c851b0db557)
4. WSO2 Products Dockerized: Remote Debug Dockerized WSO2 Products with IntelliJ IDEA [https://medium.com/faun/wso2-products-dockerized-remote-debug-dockerized-wso2-products-with-intellij-idea-111ca8ae60c](https://medium.com/faun/wso2-products-dockerized-remote-debug-dockerized-wso2-products-with-intellij-idea-111ca8ae60c)
5. WSO2 Products Dockerized: Image Tagging Mechanism [https://medium.com/faun/wso2-products-dockerized-image-tagging-mechanism-9b4499831548](https://medium.com/faun/wso2-products-dockerized-image-tagging-mechanism-9b4499831548)
6. Monitoring WSO2 Product with Prometheus

* 1. [https://medium.com/@pulasthi7/monitoring-wso2-identity-server-health-with-prometheus-grafana-886f1524930f](https://docs.wso2.com/display/EI650/Monitoring+the+ESB+Profile+with+Prometheus)

* 2. [https://medium.com/@lashan/monitoring-wso2-products-with-prometheus-4ace34759901](https://medium.com/@lashan/monitoring-wso2-products-with-prometheus-4ace34759901)

7. Monitor WSO2 product with Appdynamics

* 1. [https://medium.com/@raj10x/monitor-wso2-products-using-appdynamics-8faf72e83a7](https://medium.com/@raj10x/monitor-wso2-products-using-appdynamics-8faf72e83a7)

#
