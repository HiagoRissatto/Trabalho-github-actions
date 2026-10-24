## Automações com GitHub Actions

Este repositório tem dois workflows em `.github/workflows/`:

- **Changelog** (`changelog.yml`): roda a cada push na `master` e gera o arquivo `CHANGELOG.md`.
- **Create Release** (`release.yml`): roda quando uma tag que começa com `v` é enviada e publica uma release no GitHub.

### Changelog automático

**O que é.** O `CHANGELOG.md` é o histórico de mudanças do projeto. Ele não é escrito à mão: o workflow o gera a partir dos commits.

**Formato.** Uma linha por commit, do mais novo para o mais antigo:

```text
- mensagem do commit (autor, data)
```

**Como funciona**, passo a passo:

1. O workflow roda a cada push na `master`, ou manualmente pelo botão **Run workflow**.
2. A action `actions/checkout` baixa o repositório com `fetch-depth: 0`, para ter todo o histórico. Sem isso, só o último commit viria.
3. O comando `git log` lista os commits, ignorando os commits de merge (`--no-merges`) e os commits do próprio robô (`--invert-grep --grep="^docs: atualiza changelog"`).
4. A saída é gravada em `CHANGELOG.md`, sobrescrevendo o arquivo inteiro.
5. Se o arquivo mudou, o robô `github-actions[bot]` faz o commit `docs: atualiza changelog [skip ci]` e o push.

**Decisões importantes:**

- `permissions: contents: write` autoriza o workflow a gravar no repositório.
- `[skip ci]` na mensagem do robô evita que o commit dele dispare o workflow de novo (loop infinito).
- O arquivo é regenerado do zero, então edições manuais no `CHANGELOG.md` são perdidas.
- O changelog é tão informativo quanto as mensagens de commit. Por isso usamos prefixos: `feat:` (novidade), `fix:` (correção), `docs:` (documentação) e `style:` (aparência).

### Release

**O que é.** Uma release é uma versão publicada do projeto, ligada a uma tag do Git (por exemplo `v1.0.0`), com notas de versão.

**Como funciona:**

1. O workflow escuta o push de tags que começam com `v` (`tags: ["v*"]`). Um `git push` comum de branch não o dispara.
2. `permissions: contents: write` permite criar a release.
3. A action `softprops/action-gh-release@v2` cria a release usando o nome da tag (`github.ref_name`).
4. `generate_release_notes: true` faz o GitHub escrever as notas automaticamente, a partir dos Pull Requests mesclados desde a versão anterior.
5. `draft: false` e `prerelease: false` publicam a release imediatamente, como versão estável.

**Como criar uma release:**

```bash
git switch master
git pull origin master
git tag -a v1.0.0 -m "Primeira versao"
git push origin v1.0.0
```

Depois, confira em **Actions** (execução do *Create Release*) e na página inicial do repositório, em **Releases**.

**Versionamento.** Usamos o formato `vMAJOR.MINOR.PATCH`:

- MAJOR: mudança que quebra compatibilidade.
- MINOR: funcionalidade nova compatível.
- PATCH: correção de erro.

### Como reproduzir do zero

1. Crie a pasta `.github/workflows/` na raiz do repositório.
2. Adicione `changelog.yml` e `release.yml`.
3. Em **Settings → Actions → General → Workflow permissions**, habilite **Read and write permissions**.
4. Se a `master` tiver proteção de branch, ela não pode bloquear o push do `github-actions[bot]`.
5. Faça commits com prefixos (`feat:`, `fix:`, `docs:`) e o changelog se atualiza sozinho.
6. Crie e envie uma tag `v*` para gerar a release.


Hiago Freitas Rissatto RA:250535
Danilo Augusto Maia RA:250299
