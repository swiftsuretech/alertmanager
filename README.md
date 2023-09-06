# Enabling SMTP alerting for Prometheus Alerts

1. This repo contains the following example files:

#### test_rule.yaml

    This is a simple, example prometheus rule. The logic of the rule is what triggers the alert. In this example, the expression asserts that 2 is greater than 1, ie. for test purposes it will always trigger. Generate a suitable rule that follows the correct logic for your alert

#### alertmanager.yaml

    This is the override file for your alerting stack. In this example we have defined our SMTP server variables, template directory etc. You have already provided this by email and formatting looks correct

#### notification.tmpl

    The go templating file that sets the parameters for your email, ie, subject and message body for an alert / remediation

2. Apply the "test_rule" manifest:

    ```bash
    kubectl create -f test_rule.yaml
    ```

3. Visit the prometheus dashboard to check that the alert is picked up. Ensure that the correct labels are applied if that's not the case:

    ![prometheus](assets/prometheus.png)

4. Now we'll update our alertmanager deployment by placing both the alertmanager.yaml and its corresponding template into a secret

    ```yaml
    kubectl create secret generic -n kommander \
    alertmanager-kube-prometheus-stack-alertmanager \
    --from-file=alertmanager.yaml \
    --from-file=notification.tmpl
    ```

5. The prometheus dashboard, by this stage, should show the alert to be firing:

    ![firing](assets/prometh_firing.png)

6. Delete the alertmanager pod to reload the configuration.

    ```yaml
    kubectl delete po -n kommander alertmanager-kube-prometheus-stack-alertmanager-0
    ```

7. Inspect the alertmanager logs to troubleshoot the smtp side.

    ```yaml
    kubectl logs -f -n kommander alertmanager-kube-prometheus-stack-alertmanager-0
    ```

Using TLS authentication for your SMTP server

8. Generate a kubernetes secret with your certificates. If this secret already exists then check to ensure that it is empty. Always back up the file to your local machine.

    ```yaml
    kubectl create secret generic -n kommander \
    alertmanager-kube-prometheus-stack-alertmanager-tls-assets-0 \
    --from-file=tls.crt=path/to/your/tls.crt \
    --from-file=tls.key=path/to/your/tls.key
    ```

9. Inspect the StatefulSet called "alertmanager-kube-prometheus-stack-alertmanager" in the "kommander" namespace. It should already define a volume called "tls-assets" which it mounts at "/etc/alertmanager/certs"

10. Update your alertmanager.yaml to add the following

    ```yaml
        receivers:
        - name: 'email'
          email_configs:
          - to: 'your@email.com'
            tls_ca: /etc/alertmanager/certs/tls.crt
            tls_cert: /etc/alertmanager/certs/tls.crt
            tls_key: /etc/alertmanager/certs/tls.key
    ```

11. Delete the alertmanager pod to reload the configuration.

    ```bash
    kubectl delete po -n kommander alertmanager-kube-prometheus-stack-alertmanager-0
    ```

12. Inspect the new pod. The certificate information should be mounted correctly and available to prometheus for SMTP authentication.
