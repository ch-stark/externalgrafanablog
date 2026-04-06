# Connecting External Grafana to RHACM: An Updated Guide

A common request from Red Hat Advanced Cluster Management (RHACM) users is how to visualize fleet-wide metrics in an external, standalone Grafana instance rather than using the built-in observability dashboard.

While there is an older Red Hat blog post covering this, the procedure has aged. Changes in API routes and secret management mean that the old steps often fail without specific modifications.

In this post, we'll provide an updated method for extracting the necessary connection details and clarify the boundaries of support for this configuration.

## Understanding the Support Scope

Before we dive into the technical steps, it is vital to understand what is (and isn't) supported by Red Hat when using an external Grafana instance:

- **Supported:** The connectivity to the Observatorium API and the health of the underlying metrics pipeline.
- **Not Supported:** The configuration, maintenance, or troubleshooting of external Grafana dashboards. If you choose to use your own Grafana instance, the responsibility for dashboard persistence and visualization logic remains with the user.
- **Security Note:** The certificates extracted for this connection grant read-only access to metrics.

## Step-by-Step: Extracting Connection Credentials

To connect an external Grafana, you need four pieces of information: the API URL, the CA Certificate, the Client Certificate, and the Client Key.

The following script automates the extraction of these values from your OpenShift cluster. This assumes you have `oc` and `jq` installed and are logged into the cluster where RHACM observability is deployed.
```bash
# Create a temporary file to store the details
touch /tmp/external-grafana.txt

echo "# --- RHACM Observability Connection Details ---" > /tmp/external-grafana.txt

# 1. Fetch the Observatorium API URL
host=$(oc -n open-cluster-management-observability get route observatorium-api -o json | jq -r '.spec.host')
echo "# URL" >> /tmp/external-grafana.txt
echo "https://${host}/api/metrics/v1/default" >> /tmp/external-grafana.txt
echo >> /tmp/external-grafana.txt

# 2. Fetch the CA Certificate
crt=$(oc -n open-cluster-management-observability get secret observability-server-ca-certs -o json | jq -r '.data."ca.crt"' | base64 -d )
echo "# ca.crt" >> /tmp/external-grafana.txt
echo "${crt}" >> /tmp/external-grafana.txt
echo >> /tmp/external-grafana.txt

# 3. Fetch the Client Certificate (tls.crt)
client_crt=$(oc -n open-cluster-management-observability get secret observability-grafana-certs -o json | jq -r '.data."tls.crt"'| base64 -d)
echo "# client.crt" >> /tmp/external-grafana.txt
echo "${client_crt}" >> /tmp/external-grafana.txt
echo >> /tmp/external-grafana.txt

# 4. Fetch the Client Key (tls.key)
client_key=$(oc -n open-cluster-management-observability get secret observability-grafana-certs -o json | jq -r '.data."tls.key"' | base64 -d)
echo "# client.key" >> /tmp/external-grafana.txt
echo "${client_key}" >> /tmp/external-grafana.txt

echo "Extraction complete. View details at /tmp/external-grafana.txt"
` `` 
## Configuring the Grafana Data Source

Once you have these values, log in to your external Grafana instance and add a new Prometheus data source with the following settings:

1. **URL:** Use the URL extracted in Step 1.
2. **TLS Client Auth:** Toggle this to **On**.
3. **With CA Cert:** Toggle this to **On**.
4. **TLS Details:**
   - Paste the `ca.crt` into the **CA Cert** field.
   - Paste the `client.crt` into the **Client Cert** field.
   - Paste the `client.key` into the **Client Key** field.
5. **Server Name:** *(Optional)* You may need to provide the hostname from the URL if your Grafana instance has strict TLS verification.

---

## Summary

By using the `observability-grafana-certs` secret, you are leveraging the same credentials the internal RHACM Grafana uses, ensuring a stable, read-only path to your global metric data. Just remember: while the data is yours to explore, the external dashboard management is a "bring-your-own" endeavor!
```
