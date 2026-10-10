# Processo de Construção do Workflow

1. Para tornar a dependência da action mais precisa e evitar que uma tag móvel aponte para outro código no futuro, foi fixada a versão da action `actions/checkout` por hash SHA. 

[Regras de auditoria do zizmor - unpinned-uses](https://docs.zizmor.sh/audits/#unpinned-uses)

2. Substituído o TODO de Secrets Scanning pela action do Gitleaks, também fixada por hash, e passando secrets.GITHUB_TOKEN.

3. Adicionado `workflow_dispatch`, permitindo iniciar o workflow manualmente.

4. [O Gitleaks encontrou uma chave de API em um commit antigo](https://github.com/AngelicaCSSilva/projeto-devsecop-desafio/actions/runs/37865413933) de `src/script.js`. Apagar a chave do arquivo atual não remove o conteúdo dos commits anteriores.

```shell
Finding:     const API_KEY = "REDACTED"
Secret:      REDACTED
RuleID:      generic-api-key
Entropy:     5.003258
File:        src/script.js
Line:        1
Commit:      cd833cd2c8357fd98efa465e7288afcb50a7ef93
Author:      Matheus Farias
Email:       matheusb994@gmail.com
Date:        2026-04-15T14:59:27Z
Fingerprint: cd833cd2c8357fd98efa465e7288afcb50a7ef93:src/script.js:generic-api-key:1
Link:        https://github.com/AngelicaCSSilva/projeto-devsecop-desafio/blob/cd833cd2c8357fd98efa465e7288afcb50a7ef93/src/script.js#L1

12:33AM INF 32 commits scanned.
12:33AM DBG Note: this number might be smaller than expected due to commits with no additions
12:33AM INF scanned ~29343 bytes (29.34 KB) in 168ms
12:33AM WRN leaks found: 1
```

5. Para limpar o histórico, foi criado um arquivo temporário de substituições, fora do repositório. O `git-filter-repo` reescreveu os commits afetados, substituindo o valor exposto por `SECRET_API_KEY` e `SECRET_DB_PASSWORD`.

6. A cópia reescrita foi verificada com o Gitleaks antes do envio das branches atualizadas ao GitHub.

7. [A limpeza foi validada pelo workflow](https://github.com/AngelicaCSSilva/projeto-devsecop-desafio/actions/runs/37865805002/job/113612129961).

8. Adicionado step para verificar se as secrets estão cadastradas. 

9. Em `src/script.js`, os textos `SECRET_API_KEY` e `SECRET_DB_PASSWORD` foram mantidos como marcadores. O passo `Inserir Secrets no JavaScript` recebe `API_KEY` e `DB_PASSWORD` do GitHub Actions por `env` e usa `sed -i` para substituir os marcadores no checkout do runner. Essa alteração é feita durante a execução do workflow e não altera os commits do repositório.

10. Após a substituição, um loop Bash percorre `API_KEY` e `DB_PASSWORD`. Para cada variável, `grep -Fq` procura o valor correspondente em `src/script.js`; se não encontrar algum deles, o passo encerra com erro. A verificação confirma que a substituição ocorreu, sem imprimir os valores dos secrets nos logs.

11. Adicionado o Semgrep para executar o SAST, uma análise que procura padrões de código potencialmente inseguros. O comando analisa `src/` usando as regras `auto` e `p/xss`. Com `--error`, a pipeline falha quando o Semgrep encontra um problema.

12. [Na primeira análise](https://github.com/AngelicaCSSilva/projeto-devsecop-desafio/actions/runs/37871807731), o Semgrep encontrou o uso de `eval()` em `src/script.js`. O conteúdo digitado pelo usuário (`input.value`) é concatenado em uma string executada pelo `eval()`, o que pode permitir injeção de código. O achado foi classificado como bloqueante:

```shell
src/script.js
❯❱ javascript.browser.security.eval-detected.eval-detected
      ❰❰ Blocking ❱❱
      Detected the use of eval(). eval() can be dangerous if used to evaluate dynamic content. If this
      content can be input from outside the program, this may be a code injection vulnerability. Ensure
      evaluated content is not definable by external sources.
      Details: https://sg.run/7ope

       29┆ eval('console.log("Tarefa adicionada: ' + input.value + '")');
```

> [!NOTE]
> Ref.: https://semgrep.dev/r?q=javascript.browser.security.eval-detected.eval-detected

13. [Correção validada pelo workflow](https://github.com/AngelicaCSSilva/projeto-devsecop-desafio/actions/runs/37872696239). 

14. A action Gitleaks fixada no workflow (`gitleaks/gitleaks-action` v3.0.0) apresenta uma limitação documentada no [issue #236](https://github.com/gitleaks/gitleaks-action/issues/236): nos eventos `push` e `pull_request`, são usados os argumentos `--no-merges --first-parent` no intervalo do `git log`, sem uma opção para substituí-los. Com `--first-parent`, commits alcançáveis apenas pelo segundo pai de um merge podem ficar fora da análise; com `--no-merges`, os próprios commits de merge são excluídos. Como consequência, o job pode terminar com sucesso sem examinar todo o conteúdo introduzido pelo merge. A ausência de achados nessa execução não comprova que o repositório esteja livre de segredos. Neste workflow, o gatilho configurado é `push` para `main`; portanto, o merge do PR gera o push que executa a análise.

Como alternativa, recomenda-se substituir a action pelo Gitleaks CLI e definir explicitamente o escopo da análise. Para examinar o histórico completo do repositório, pode ser usado o seguinte comando:

```bash
docker run --rm \
  -v "${{ github.workspace }}:/repo" \
  -w /repo \
  ghcr.io/gitleaks/gitleaks:<versao> \
  git --verbose --redact .
```

Para analisar apenas um intervalo de commits, usa-se `LOG_OPTS="--all $BASE_SHA..$HEAD_SHA"`

15. Versão da imagem do GitLeaks fixada por SHA (v8.30.1), evitando o uso de `latest` ou tags.

16. Com os novos trigger, as PRs começam a disparar o fluxo. Até a finalização do workflow, a ruleset bloqueante não será ativada.

17. A validação dos secrets obrigatórios foi isolada no job `verify`. O job `build` depende dessa validação e mantém a etapa de build como simulação.

18. Gitleaks, Semgrep e Grype foram separados em jobs com checkout próprio. As três análises executam em paralelo após o job `build`.

19. Adicionado o SCA com Grype, precedido pela geração do `package-lock.json`. 

20. O empacotamento, o cálculo do hash, a assinatura demonstrativa e o upload foram movidos para o job `package-artifact`, condicionado ao sucesso das três análises.

21. O deploy depende do sucesso do empacotamento do artefato.
