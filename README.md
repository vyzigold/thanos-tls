# Thanos TLS

Steps I took to make this work:
- Deploy OCP + OSP by following the readme in: https://github.com/openstack-k8s-operators/telemetry-operator
- Install COO or OBO
- Execute `oc edit oscp` and enable "metricstorage" under telemetry

This will create a MonitoringStack caled "metric-storage" with Prometheus and AlertManager. Prometheus web endpoint is behind TLS by default.

- Create a new Thanos Querier by executing `oc apply -f thanos.yaml`. Notice, that the data from Prometheus are accessible in Thanos even without configuring anything. The Thanos sidecar is serving the data in plain text.

- Use cert-manager automatically installed with OSP to generate certificates by executing: `oc apply -f cert.yaml` and `oc apply -f cert_web.yaml`

- Scale down the observability-operator deployment to prevent it from interveneing with our changes: `oc scale --replicas=0 -n openshift-operators deploy/observability-operator`

- Create a ConfigMap with configuration needed to secure Thanos's web endpoint with: `oc create cm --from-file=http.conf thanos-http-conf`

- Look at the prometheus\_with\_thanos\_tls.yaml on how to change the Prometheus resource to secure the thanos-sidecar grpc endpoint with TLS. Mainly notice the spec.thanos and spec.volumes sections.

- Look at the thanos\_deployment.yaml on how to change the Thanos querier deployment so that it can access the thanos sidecar with TLS and so that the web endpoint uses TLS. Notice mainly the spec.template.spec.containers.args, spec.template.spec.containers.volumeMounts and spec.template.spec.volumes.



Now you should be able to see the Prometheus data and Thanos should be using TLS for its connections.
