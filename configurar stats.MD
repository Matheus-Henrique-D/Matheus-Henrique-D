# Como gerar as estatísticas do GitHub com Actions

## Por que as linguagens não estavam aparecendo?

Existem três motivos principais para as linguagens terem ficado zeradas anteriormente:
1. **Forks desativados por padrão:** Projetos importantes (como *NutriAlerta* e *SentinelaAI*) são forks no seu perfil. Por padrão, a action ignora forks a menos que `repositories_forks: yes` esteja habilitado.
2. **Escopo do token padrão (`GITHUB_TOKEN`):** O token automático que o GitHub fornece para a action roda apenas no contexto do repositório local e muitas vezes não tem permissão para ler metadados detalhados de linguagens de outros repositórios e forks.
3. **Falta de associação explícita (`repositories_affiliations` e `user`):** Especificando explicitamente o usuário e que devem ser contados repositórios próprios e colaborações.

---

## O que foi ajustado no Workflow (`.github/workflows/main.yml`)

O workflow foi atualizado para:
- Incluir `repositories_forks: yes` (para computar projetos em forks);
- Suportar tanto um token com maiores privilégios (`METRICS_TOKEN`) quanto o padrão (`GITHUB_TOKEN`);
- Exibir detalhes extras de porcentagem e tamanho em bytes (`plugin_languages_details: percentage, bytes-size`);
- Aumentar o timeout de análise para 30 segundos.

---

## Recomendado: Como criar o `METRICS_TOKEN` (Garante 100% dos dados)

Para que a action consiga ler todas as linguagens de repositórios públicos, privados e forks sem restrições da API:

1. Vá em [GitHub Personal Access Tokens (Classic)](https://github.com/settings/tokens) (ou acerte em: **Avatar** > **Settings** > **Developer Settings** > **Personal access tokens** > **Tokens (classic)**).
2. Clique em **Generate new token** -> **Generate new token (classic)**.
3. No campo **Note**, coloque algo como `Metrics Profile`.
4. Marque os escopos:
   - `repo` (para ler código e repositórios);
   - `read:user` (para ler dados do perfil).
5. Clique em **Generate token** e copie o token gerado.
6. Vá no seu repositório de perfil: **Matheus-Henrique-D/Matheus-Henrique-D**.
7. Vá em **Settings** > **Secrets and variables** > **Actions**.
8. Clique em **New repository secret**:
   - Nome: `METRICS_TOKEN`
   - Valor: Cole o token que você copiou.
9. Vá na aba **Actions** > selecione **Generate GitHub metrics** > clique em **Run workflow**.

*(Nota: caso você não crie o secret `METRICS_TOKEN`, o workflow usará o `GITHUB_TOKEN` como fallback).*
