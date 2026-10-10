# Desafio DevSecOps — Gerenciador de Tarefas

## Sobre o Projeto
Este repositório faz parte do desafio prático do módulo de DevSecOps da ADA Tech.
Você receberá este projeto com vulnerabilidades propositais e uma pipeline incompleta.
Seu objetivo é **implementar a pipeline de segurança** e **corrigir as vulnerabilidades**.

## Estado atual
A pipeline está **incompleta**. Os steps de segurança precisam ser implementados por você.

## Sua missão
1. Implementar os steps de segurança no `pipeline.yml`
2. Fazer a pipeline **quebrar** ao detectar os problemas
3. Corrigir as vulnerabilidades encontradas
4. Fazer a pipeline **passar** com tudo verde ✅
5. Documentar o funcionamento da pipeline neste README

## O que implementar
- [ ] Secrets Scanning com **Gitleaks**
- [ ] SAST com **Semgrep**
- [ ] SCA com **Grype**
- [ ] Assinatura do artefato com cosign
- [ ] Deploy com **GitHub Pages**

## Como a pipeline funciona
> [!NOTE]
> O histórico de como este workflow foi montado está em `docs/processo.md`.

O workflow criado realiza as seguintes etapas. Depois do Build, as análises com Gitleaks, Semgrep e Grype rodam em paralelo:

1. **O GitHub prepara o projeto.** Ele baixa uma cópia dos arquivos e do histórico do repositório. A versão da ferramenta de checkout foi fixada para que o workflow use sempre a mesma versão.

2. **Confere os valores necessários.** `API_KEY` e `DB_PASSWORD` são valores cadastrados na área de secrets do GitHub, fora dos arquivos do projeto. O workflow confere se ambos existem e não estão vazios. Se faltar algum, para e mostra qual precisa ser cadastrado, sem revelar seu conteúdo.

3. **Preenche e confere os dados sensíveis.** `src/script.js` contém marcadores no lugar desses dados. Durante a execução, a pipeline substitui os marcadores pelos valores cadastrados como Secrets e confere se foram inseridos. Se algum dado não for inserido, a etapa falha.

4. **Faz uma conferência básica.** A etapa chamada “Build” só lista os arquivos e escreve “Build OK”. Ela ainda não constrói nem testa o site, servindo apenas como uma etapa ilustrativa.

5. **Procura credenciais expostas com Gitleaks.** Ele verifica os commits incluídos na execução, procurando valores que pareçam senhas ou chaves salvos no Git. Essa busca não confere o arquivo que foi alterado temporariamente durante a execução.

6. **Analisa o código com Semgrep.** Essa ferramenta procura padrões que podem indicar falhas de segurança. A opção `--error` faz essa etapa falhar se a análise encontrar problemas.

7. **Verifica as dependências com Grype (SCA).** A pipeline gera uma lista das dependências do projeto e procura vulnerabilidades conhecidas nelas. Se encontrar alguma de nível médio ou mais alto, a etapa falha.

## URL de Produção
> Adicione aqui o link do GitHub Pages após o deploy.
