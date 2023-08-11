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

    ```bash
    kubectl create secret generic -n kommander \
    alertmanager-kube-prometheus-stack-alertmanager \
    --from-file=alertmanager.yaml \
    --from-file=notification.tmpl
    ```

5. Delete the alertmanager pod to reload the configuration.
