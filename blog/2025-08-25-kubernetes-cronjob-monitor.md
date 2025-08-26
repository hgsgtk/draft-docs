# Effective Kubernetes CronJob Monitoring with kube-state-metrics

## Introduction

Kubernetes CronJobs are essential for running scheduled tasks in production environments, but monitoring their health and execution status can be challenging. Traditional monitoring approaches often miss critical issues like misconfigurations, failed executions, or jobs that never start. This is where kube-state-metrics comes in - a powerful tool that generates comprehensive metrics about Kubernetes objects including CronJobs, Jobs, and pods.

In this post, I'll show you how to implement effective CronJob monitoring using kube-state-metrics and Prometheus, covering both success-based and failure-based monitoring strategies.

## Understanding kube-state-metrics

[kube-state-metrics](https://github.com/kubernetes/kube-state-metrics) is a service that generates metrics from the Kubernetes API server, providing detailed insights into the state of various Kubernetes objects. For CronJob monitoring, it generates several key metrics that help us understand job execution status:

- **kube_job_status_succeeded**: The number of pods that reached Phase Succeeded
- **kube_job_status_failed**: The number of pods that reached Phase Failed and the reason for failure  
- **kube_job_complete**: Whether the job has completed its execution
- **kube_job_failed**: Whether the job has failed its execution

These metrics give us visibility into both successful and failed job executions, allowing us to build comprehensive monitoring solutions.

## Monitoring Strategy 1: Success-based Approach

The most straightforward approach to monitoring CronJob health is using the `kube_job_status_succeeded` metric. This approach focuses on detecting when jobs are not running as expected, including scenarios where Kubernetes fails to spin up pods due to misconfiguration.

Here's the PromQL query for this monitoring strategy:

```promql
sum(kube_job_status_succeeded{namespace="<namespace>", job_name=~"<job_name>.*"} >= 0) OR vector(0)
```

The key insight here is the `OR vector(0)` clause. When a Kubernetes CronJob is misconfigured (for example, with an invalid Docker registry URL), there might be no data available for the metric. Without this fallback, your monitoring system could miss critical failures. This approach correctly captures error cases where Kubernetes fails to create pods entirely.

### Setting Up Alerts

To create actionable monitoring, you'll want to set up alerts in Grafana or Alertmanager. Here's how to configure an alert for the success-based monitoring approach:

**Alert Configuration:**
- **Data source**: Prometheus
- **PromQL**: `sum by() (kube_job_status_succeeded{namespace="default", job_name=~"sample-cronjob.*"}) >= 0 OR vector(0)`
- **Alert condition**: `IS BELOW OR EQUAL TO` 0

This alert will trigger when the CronJob hasn't had any successful executions, indicating either a misconfiguration or that the job hasn't run yet. The `sum by()` aggregation groups the results, and the `OR vector(0)` ensures the alert remains active even when there's no data.

## Monitoring Strategy 2: Failure-based Approach

An alternative monitoring strategy uses the `kube_job_failed` metric to track job failures directly. This approach is particularly useful when you need to monitor for specific failure conditions.

The `kube_job_failed` metric is influenced by several Job configuration parameters:

- **successfulJobsHistoryLimit**: Controls how many successful jobs are retained
- **failedJobsHistoryLimit**: Controls how many failed jobs are retained  
- **ttlSecondsAfterFinished**: Automatically cleans up old jobs after completion (stable since Kubernetes 1.23)

Since `kube_job_failed` represents the number of failed jobs, its value will always be 1 without proper TTL settings. This makes it ideal for scenarios where any single failure is critical. To implement this strategy effectively, you'll need to:

1. Set `ttlSecondsAfterFinished` in your Job specifications
2. Create an alert rule: `kube_job_failed > 0`

## Setting up kube-state-metrics

The prometheus-community Helm charts provide an excellent way to deploy the Prometheus stack in your Kubernetes cluster. Let's start with the basic setup:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

For a minimal setup, you can deploy just kube-state-metrics:

```bash
helm install prometheus-community/kube-state-metrics --generate-name
```

This will deploy kube-state-metrics in the default namespace. You can specify a different namespace using the `--namespace <namespace>` flag.

Verify the deployment:

```bash
$ helm list
NAME                         	NAMESPACE	REVISION	UPDATED                             	STATUS  	CHART                   	APP VERSION
kube-state-metrics-1756127123	default  	1       	2025-08-25 22:05:25.775835 +0900 JST	deployed	kube-state-metrics-6.1.4	2.16.0
```

Test the metrics endpoint:

```bash
kubectl run tmp-curl --rm -i --tty \
  --image=curlimages/curl:8.8.0 \
  --restart=Never \
  -- curl -v http://kube-state-metrics-1756127123.default.svc.cluster.local:8080/metrics
```

## Complete Prometheus Stack Deployment

For production environments, you'll want the complete Prometheus monitoring stack. Use the `prometheus-community/kube-prometheus-stack` chart:

```bash
helm install prometheus prometheus-community/kube-prometheus-stack --namespace monitoring
```

This chart includes Prometheus, Alertmanager, and kube-state-metrics. Test the metrics endpoint:

```bash
kubectl run tmp-curl --rm -i --tty \
  --image=curlimages/curl:8.8.0 \
  --restart=Never \
  --namespace monitoring \
  -- curl -v http://prometheus-kube-state-metrics.monitoring.svc.cluster.local:8080/metrics
```

The output will show various metrics including:

```
kube_job_status_active{namespace="default",job_name="job-1"} 0
kube_job_status_active{namespace="default",job_name="sample-cronjob-29269500"} 0
kube_job_status_active{namespace="default",job_name="sample-cronjob-29269499"} 0
kube_job_status_active{namespace="default",job_name="sample-cronjob-29269498"} 0
kube_job_status_active{namespace="default",job_name="broken-cronjob-29268755"} 0
# HELP kube_job_complete [STABLE] The job has completed its execution.
# TYPE kube_job_complete gauge
kube_job_complete{namespace="default",job_name="job-1",condition="true"} 1
kube_job_complete{namespace="default",job_name="job-1",condition="false"} 0
```

## Advanced Setup with Grafana

For better visualization and dashboard capabilities, you can add Grafana to your monitoring stack:

```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

kubectl create namespace grafana
helm install grafana grafana/grafana --namespace grafana
```

Access Grafana through port forwarding:

```bash
kubectl --namespace grafana port-forward grafana-6b967df7bc-xkkjs 3000
```

Then open your browser and navigate to `http://localhost:3000`.

## Accessing Prometheus and Grafana

### Prometheus Access

Access the Prometheus server through port forwarding:

```bash
kubectl port-forward -n monitoring svc/prometheus-operated 9090:9090
```

### Grafana Configuration

The default Grafana credentials are:
- Username: `admin`
- Password: Retrieve using this command:

```bash
kubectl get secret --namespace grafana grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
```

Once logged in, add Prometheus as a data source:
- Type: `Prometheus`
- URL: `http://prometheus-operated.monitoring.svc.cluster.local:9090`

## Best Practices and Considerations

When implementing CronJob monitoring, consider these key points:

- **Choose your monitoring strategy wisely**: Success-based monitoring is better for catching misconfigurations, while failure-based monitoring is ideal for critical failure scenarios
- **Handle edge cases**: Always use fallback values like `OR vector(0)` to ensure your monitoring system remains active
- **Set appropriate TTLs**: Configure `ttlSecondsAfterFinished` to maintain clean job history
- **Namespace isolation**: Deploy monitoring components in dedicated namespaces for better organization
- **Resource limits**: Set appropriate resource requests and limits for monitoring components

## Conclusion

Effective Kubernetes CronJob monitoring requires understanding both the metrics available and the monitoring strategies that best fit your use case. By leveraging kube-state-metrics with either success-based or failure-based monitoring approaches, you can build robust monitoring systems that catch both obvious failures and subtle misconfigurations.

The combination of kube-state-metrics, Prometheus, and Grafana provides a powerful foundation for monitoring Kubernetes workloads. Start with the basic setup and gradually expand to the full stack as your monitoring needs grow.

Remember that monitoring is not just about collecting metrics - it's about creating actionable insights that help you maintain reliable, production-ready Kubernetes clusters.
