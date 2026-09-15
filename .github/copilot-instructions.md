# Instruções de review — th0th

Monorepo TypeScript com **Bun + Turborepo**. `apps/tools-api`, `apps/mcp-client`,
`apps/opencode-plugin`, `packages/core`, `packages/shared`. Publica pacotes.

## Rejeite o PR se

- **Segredo no diff** ou `.env` versionado (existe `.env.example` para isso).
- **`npm`/`yarn`/`pnpm` introduzido.** O gerenciador é Bun e o lockfile é
  `bun.lock`. Lockfile de outro gerenciador no diff é erro.
  Atenção: o `package.json` declara `packageManager: bun@1.2.0`, mas o CI usa
  `oven-sh/setup-bun` com `bun-version: latest` — a versão **não é imposta**.
  Divergência de comportamento entre local e CI pode vir daí.
- **Dependência entre workspaces sem entrar no `package.json`** do pacote que
  consome — o Turbo monta o grafo por ali; sem isso o build passa local e quebra
  no CI (que roda `--frozen-lockfile`).
- **Contrato público de `packages/core` ou `packages/shared` mudado sem bump de
  versão.** São publicados; quebra silenciosa vaza para consumidor.
- **`any` novo em API exportada.**

## Sinalize

- Script novo que não passa por `turbo` (perde cache e paralelismo).
- Só `packages/core` define `test`; `apps/*` não tem nenhum. Rota ou
  comportamento novo em `apps/*` chega sem cobertura — vale pedir teste.
- Rota nova no `tools-api` sem entrada no Swagger — o CI faz smoke test em
  `/swagger`.
- Mudança em `packages/core` que afeta busca/ranking sem rodar bench
  (`bench:fixture:sicad`, `bench:beir`).

## Comandos

```bash
bun install --frozen-lockfile
bun run type-check && bun run build && bun run test   # a ordem do CI
```

⚠️ `bun run lint` **não verifica nada hoje**: nenhum `package.json` de
`apps/*` ou `packages/*` define script `lint`, então `turbo run lint` passa
vazio. Não trate como gate — e não sinalize um PR por "não passar no lint".
Adicionar lint de verdade nos workspaces é trabalho pendente.
