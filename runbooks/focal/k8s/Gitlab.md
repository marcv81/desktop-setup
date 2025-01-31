# GitLab

Deploy the Helm chart.

    helm repo add gitlab https://charts.gitlab.io/
    helm repo update
    helm --namespace gitlab install gitlab gitlab/gitlab \
        --timeout 600s \
        --set global.edition=ce \
        --set global.hosts.domain=instantra.men \
        --set certmanager.install=false \
        --set global.ingress.configureCertmanager=false \
        --set global.ingress.annotations."kubernetes\.io/tls-acme"=true \
        --set gitlab.webservice.ingress.tls.secretName=instantramen-tls \
        --set registry.ingress.tls.secretName=instantramen-tls \
        --set minio.ingress.tls.secretName=instantramen-tls \
        --set gitlab.kas.ingress.tls.secretName=instantramen-tls

Find the external IP address of the ingress controller.

    kubectl get service gitlab-nginx-ingress-controller

Add the domain name to `/etc/hosts`.

    172.18.0.3 gitlab.localdomain

Change the root password.

    kubectl exec -it gitlab-toolbox-5b8b5555df-w75fl -c toolbox -- bash
    gitlab-rake "gitlab:password:reset[root]"
