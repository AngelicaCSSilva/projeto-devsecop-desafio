# Processo de Construção do Workflow

1. Para tornar a dependência da action mais precisa e evitar que uma tag móvel aponte para outro código no futuro, foi fixada a versão da action `actions/checkout` por hash SHA. 

[Regras de auditoria do zizmor - unpinned-uses](https://docs.zizmor.sh/audits/#unpinned-uses)

2. Substituído o TODO de Secrets Scanning pela action do Gitleaks, também fixada por hash, e passando secrets.GITHUB_TOKEN.

3. Adicionado `workflow_dispatch`, permitindo iniciar o workflow manualmente.

