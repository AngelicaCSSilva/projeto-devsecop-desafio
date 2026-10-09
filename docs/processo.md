# Processo de Construção do Workflow

1. Para tornar a dependência da action mais precisa e evitar que uma tag móvel aponte para outro código no futuro, foi fixada a versão da action `actions/checkout` por hash SHA. 

[Regras de auditoria do zizmor - unpinned-uses](https://docs.zizmor.sh/audits/#unpinned-uses)

2. Substituído o TODO de Secrets Scanning pela action do Gitleaks, também fixada por hash, e passando secrets.GITHUB_TOKEN.

3. Adicionado `workflow_dispatch`, permitindo iniciar o workflow manualmente.

4. O Gitleaks encontrou uma chave de API em um commit antigo de `src/script.js`. Apagar a chave do arquivo atual não remove o conteúdo dos commits anteriores.

5. Para limpar o histórico, foi criado um arquivo temporário de substituições, fora do repositório. O `git-filter-repo` reescreveu os commits afetados, substituindo o valor exposto por `SECRET_API_KEY` e `SECRET_DB_PASSWORD`.

6. A cópia reescrita foi verificada com o Gitleaks antes do envio das branches atualizadas ao GitHub.

7. Adicionado step para verificar se as secrets estão cadastradas. 