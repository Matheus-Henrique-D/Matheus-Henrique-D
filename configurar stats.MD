# Como gerar as estatísticas do GitHub com Actions

## O problema

Os cards de estatísticas e de linguagens do README usam o serviço `github-readme-stats.vercel.app`. Esse serviço é uma instância pública gratuita, compartilhada por milhares de perfis, e por isso vive estourando o limite de requisições da API do GitHub ou saindo do ar. Quando isso acontece, a imagem simplesmente não carrega — e não há parâmetro na URL que resolva, porque o problema está no servidor, não no seu README.

## A solução

Em vez de depender de um servidor de terceiros, a ideia é gerar as imagens dentro do seu próprio repositório, usando GitHub Actions. O workflow roda periodicamente, consulta a API do GitHub com o seu próprio token e salva o resultado como um arquivo SVG no repositório. O README passa a apontar para esse arquivo local, que sempre existe e sempre carrega.

A ferramenta usada é a `lowlighter/metrics`, que gera estatísticas gerais e distribuição de linguagens em uma única imagem.

## Passo 1 — Repositório de perfil

O workflow vai no repositório especial cujo nome é idêntico ao seu usuário: `Matheus-Henrique-D/Matheus-Henrique-D`. É esse repositório que o GitHub usa para montar o README exibido no seu perfil. Se ele já existe, use o mesmo; se não, crie como público.

## Passo 2 — Criar o workflow

Dentro do repositório, crie o arquivo `.github/workflows/main.yml` com o conteúdo abaixo:

```yaml
name: Generate GitHub metrics

on:
  schedule:
    - cron: "0 */12 * * *"
  workflow_dispatch:
  push:
    branches: [main]

jobs:
  metrics:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: lowlighter/metrics@latest
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          filename: github-metrics.svg
          plugin_languages: yes
          plugin_languages_analysis_timeout: 15
          plugin_languages_categories: markup, programming
```

Vale explicar as partes principais. O bloco `on` define quando o workflow roda: a cada 12 horas pelo `cron`, manualmente pelo `workflow_dispatch`, e a cada push na branch principal. O `permissions: contents: write` é obrigatório, porque a action precisa gravar o SVG de volta no repositório. O `secrets.GITHUB_TOKEN` é fornecido automaticamente pelo GitHub — você não precisa criar nem configurar nada. Já `plugin_languages` é o que habilita o gráfico de distribuição de linguagens, justamente o que estava faltando.

## Passo 3 — Rodar pela primeira vez

Abra a aba **Actions** do repositório. Se for a primeira vez, o GitHub vai pedir para habilitar os workflows — confirme. Depois selecione o workflow "Generate GitHub metrics" na lista lateral e use o botão **Run workflow** para executá-lo manualmente, sem esperar o agendamento.

Se tudo correr bem, ao final da execução o arquivo `github-metrics.svg` aparece na raiz do repositório.

## Passo 4 — Atualizar o README

Remova os dois blocos antigos que apontavam para `github-readme-stats.vercel.app` (o de estatísticas gerais e o de top languages) e substitua por um único bloco:

```markdown
<div align="center">
  <img src="https://raw.githubusercontent.com/Matheus-Henrique-D/Matheus-Henrique-D/main/github-metrics.svg" alt="GitHub Metrics" />
</div>
```

Como a `metrics` já junta estatísticas e linguagens na mesma imagem, um bloco substitui os dois anteriores. Se a sua branch principal se chamar `master` em vez de `main`, ajuste a URL e também o `branches` do workflow.

## Observações

A imagem é atualizada a cada 12 horas, então mudanças recentes nos seus repositórios levam algumas horas para aparecer — o que é normal e não indica erro. Caso queira ajustar visual ou adicionar mais informações, a `lowlighter/metrics` tem dezenas de plugins opcionais documentados no repositório oficial da action.

Um detalhe sobre o gráfico de linguagens: por padrão ele não conta repositórios que são forks. Se boa parte do seu código estiver em forks, a distribuição pode parecer menor do que você espera — nesse caso existe configuração específica na documentação da action para incluí-los.