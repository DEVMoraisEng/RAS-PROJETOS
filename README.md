# RAS PROJETOS

Reunião de Alinhamento Semanal do Departamento de Projetos e Desenvolvimento — Morais Engenharia e Construção.

Mesma arquitetura da RAS geral / RAS Financeiro: Python lê o Notion via GitHub Actions, publica JSON estático em `dist/`, e o `index.html` consome esse JSON. Nenhum token aparece no navegador.

**Banco de dados:** ATIVIDADES RAS PROJETOS — `3bfc5ab5-32d3-803f-b1e0-eb3066f262ff`

**Setores (pessoas):** Felipe (conta Notion: `felipe berçan`), José Arthur.

## Passos para colocar no ar

### 1. Criar o repositório

Suba estes arquivos em um repositório novo (`RAS-PROJETOS`), branch `main`. Confirme que os workflows ficaram em `.github/workflows/`.

### 2. Secret

Em *Settings → Secrets and variables → Actions*, criar `NOTION_TOKEN` com o token da integração (pode ser o mesmo já usado no RAS-FINANCEIRO, desde que a integração tenha acesso a este banco também).

### 3. Compartilhar o banco com a integração

No Notion, abrir o banco ATIVIDADES RAS PROJETOS → `···` → *Conexões* → adicionar a integração.

### 4. Conferir as opções do Notion

```bash
export NOTION_TOKEN="..."
python3 verificar_status.py
```

O site espera os Status: **A Fazer, Em Andamento, Pendente, Concluído, Continuidade da Semana Anterior**. Se a coluna Status for do tipo `status` (não `select`), todas essas opções precisam existir no Notion com esse texto exato — a API não cria opção nova nesse tipo de coluna.

### 5. Primeiro fetch

*Actions → RAS Projetos - atualizar dados → Run workflow.* Gera `dist/data_atividades.json`.

### 6. GitHub Pages

*Settings → Pages → Source: **Deploy from a branch*** (não "GitHub Actions" — esse é o erro mais comum, dá 404 na hora de abrir o link) → Branch `main`, pasta `/ (root)` → Save.

### 7. Escrita no Notion (Apps Script)

`Code.gs` já está neste repositório. Passos:

1. script.google.com → *Novo projeto* → colar o conteúdo de `Code.gs`.
2. Propriedades do script (⚙️ Configurações do projeto → Propriedades do script):

| Propriedade | Valor |
|---|---|
| `NOTION_TOKEN` | o token da integração |
| `RAS_PROJ_DB_ID` | `3bfc5ab532d3803fb1e0eb3066f262ff` |
| `GITHUB_REPO` | `DEVMoraisEng/RAS-PROJETOS` |
| `GITHUB_TOKEN` | Personal Access Token do GitHub com escopo `repo` (pode reaproveitar o do RAS-FINANCEIRO se ele também tiver acesso a este repo) |

3. **PEOPLE_IDS** — ainda vazio no `Code.gs`. A coluna Responsável é do tipo Pessoa; sem o ID do usuário do Notion, o campo fica em branco na gravação. Pra preencher:
   - No Notion, preencha o Responsável de pelo menos uma linha por pessoa (Felipe, José Arthur).
   - Rode `extrair_ids_responsavel.py` (dê duplo clique nele — ele pede o token na tela e não fecha sozinho).
   - Cole o bloco `PEOPLE_IDS` impresso no `Code.gs`.

4. Implantar → Nova implantação → tipo **App da Web** → Executar como **Eu** → Quem pode acessar: **Qualquer pessoa**.

   ⚠️ **Não escolha "Somente eu"** — foi exatamente esse erro que causou horas de "invalid_request_url" no RAS-FINANCEIRO: com acesso restrito, o `fetch` do navegador esbarra numa tela de login do Google em vez de rodar o `doGet`.

5. Copie a URL `/exec` e cole em `const WRITE_ENDPOINT = "";` no `index.html`. Suba o arquivo atualizado.

6. Toda vez que editar o `Code.gs` depois disso: *Gerenciar implantações → lápis → Nova versão* (nunca "Nova implantação" — muda a URL e o site para de escrever).

7. Testar: rode `testeConexao` no editor (lista as colunas do banco) e depois `testeUpdateReal` (edite o `idReal` no código pra um ID de atividade real, pego do `dist/data_atividades.json` publicado).

## Arquivos

| Arquivo | Função |
|---|---|
| `index.html` | O site (single file) |
| `Code.gs` | Apps Script — proxy de escrita para o Notion (cole em script.google.com) |
| `fetch_ras.py` | Lê o Notion → gera `dist/data_atividades.json` |
| `rollover_semana.py` | Domingo à noite: empurra o não concluído pra semana nova com status "Continuidade da Semana Anterior" |
| `verificar_status.py` | Compara as opções do site com as do Notion (só lê, não altera) |
| `extrair_ids_responsavel.py` | Extrai os IDs de usuário do Notion pro `PEOPLE_IDS` do Apps Script |
