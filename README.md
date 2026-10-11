# Desafio DevSecOps — Gerenciador de Tarefas

## Sobre o Projeto
Este repositório faz parte do desafio prático do módulo de DevSecOps da ADA Tech.
Você receberá este projeto com vulnerabilidades propositais e uma pipeline incompleta.
Seu objetivo é **implementar a pipeline de segurança** e **corrigir as vulnerabilidades**.

## Estado atual
✅ A pipeline está **completa**.

## Sua missão
1. Implementar os steps de segurança no `pipeline.yml`
2. Fazer a pipeline **quebrar** ao detectar os problemas
3. Corrigir as vulnerabilidades encontradas
4. Fazer a pipeline **passar** com tudo verde ✅
5. Documentar o funcionamento da pipeline neste README

## O que implementar
- [X] Secrets Scanning com **Gitleaks**
- [X] SAST com **Semgrep**
- [X] SCA com **Grype**
- [X] Assinatura do artefato com cosign
- [X] Deploy com **GitHub Pages**

## Como a pipeline funciona
> [!NOTE]
> O histórico de como este workflow foi montado está em `docs/processo.md`.

O workflow criado realiza as seguintes etapas. Depois do Build, as análises com Gitleaks, Semgrep e Grype rodam em paralelo:

1. **O GitHub prepara o projeto.** Ele baixa uma cópia dos arquivos e do histórico do repositório. A versão da ferramenta de checkout foi fixada para que o workflow use sempre a mesma versão.

2. **Confere os valores necessários.** `API_KEY` e `DB_PASSWORD` são valores cadastrados na área de secrets do GitHub, fora dos arquivos do projeto. O workflow confere se ambos existem e não estão vazios. Se faltar algum, para e mostra qual precisa ser cadastrado, sem revelar seu conteúdo.

3. **Preenche e confere os dados sensíveis.** `src/script.js` contém marcadores no lugar desses dados. Durante a execução, a pipeline substitui os marcadores pelos valores cadastrados como Secrets e confere se foram inseridos. Se algum dado não for inserido, a etapa falha.

4. **Faz uma conferência básica.** A etapa chamada “Build” só lista os arquivos e escreve “Build OK”. Ela ainda não constrói nem testa o site, servindo apenas como uma etapa ilustrativa.

5. **Procura credenciais expostas com Gitleaks.** Ele verifica os commits incluídos na execução, procurando valores que pareçam senhas ou chaves salvos no Git. Essa busca não confere o arquivo que foi alterado temporariamente durante a execução.
   > **Em palavras simples:** procura senhas e chaves que foram salvas no histórico do projeto.

6. **Analisa o código com Semgrep.** Essa ferramenta procura padrões que podem indicar falhas de segurança. A opção `--error` faz essa etapa falhar se a análise encontrar problemas.
   > **Em palavras simples:** procura no código trechos que possam abrir brechas de segurança.

7. **Verifica as dependências com Grype (SCA).** A pipeline gera uma lista das dependências do projeto e procura vulnerabilidades conhecidas nelas. Se encontrar alguma de nível médio ou mais alto, a etapa falha.
   > **Em palavras simples:** confere se as bibliotecas usadas pelo projeto têm falhas conhecidas.

8. **Empacota o site e calcula seu hash.** Se as análises forem aprovadas, os arquivos do site são reunidos em um pacote. A pipeline calcula uma impressão digital (hash) desse pacote e a guarda em um arquivo separado.

9. **Assina o pacote com Cosign.** O Cosign cria uma assinatura digital usando uma identidade temporária fornecida pelo GitHub Actions. Não é necessário guardar uma chave privada permanente no repositório. A assinatura é enviada junto com o pacote.
   > **Em palavras simples:** cria um comprovante digital que identifica o workflow que assinou o pacote.

10. **Confere a assinatura antes de publicar.** No push para a branch `main`, o workflow confere se o pacote continua igual ao que foi assinado e se a assinatura corresponde a uma identidade esperada do GitHub Actions neste repositório. Se a conferência falhar, o deploy é interrompido. Se passar, o site é extraído e publicado no GitHub Pages.
    > **Em palavras simples:** confirma que o pacote não mudou e que a assinatura é esperada antes de publicar.

## URL de Produção
> [Link para o projeto no Github Pages](https://angelicacssilva.github.io/projeto-devsecop-desafio/)
