# API do GD — Lojas Ricardo's · Documentação

API em Google Apps Script que lê e grava nas planilhas GD das lojas
(**PORTO BELO, IMPERIAL, MEGA CENTER**), anos **2025** e **2026**.
Serve como fonte de dados para o ERP e para dashboards.

---

## Base

- **URL base:** `https://SEU-APP/exec` (a URL da implantação, terminada em `/exec`)
- **Autenticação:** parâmetro `token` em toda chamada. Sem token válido, a resposta é
  `{ "erro": "token inválido ou ausente" }`.
- **Formato:** JSON (UTF-8).
- **Leitura:** `GET` com `?view=...`. **Escrita:** `POST` com corpo JSON.
- **Segurança:** o token dá acesso a todo o financeiro. Guarde-o como *secret*; de preferência,
  chame a API pelo servidor (não expor o token no navegador). Para trocar o token, edite a
  constante `TOKEN` no script e reimplante uma **nova versão** da mesma implantação.

---

## Blocos disponíveis

Cada "bloco" é uma seção do GD. Use a `chave` nos filtros.

| chave | título |
|---|---|
| `vendas` | Vendas diárias |
| `vg_venda` | Vendas por grupos – preço venda |
| `vg_repo` | Vendas por grupos – reposição |
| `margem` | Margem de lucro por grupos |
| `compras` | Compras por grupos |
| `estoque` | Estoque por grupos |
| `receber` | A receber |
| `pagar` | A pagar |
| `entradas` | Entradas geral |
| `saidas` | Saídas geral |
| `emprestimos` | Empréstimos |
| `fiscal` | Movimentação fiscal |
| `impostos` | Impostos |
| `despesas` | Despesas |

Meses (colunas): `JAN FEV MAR ABR MAI JUN JUL AGO SET OUT NOV DEZ`.

---

## Endpoints de leitura (GET)

### `?view=meta`
Estrutura disponível — use para orientar a aplicação antes de puxar dados.

```
GET /exec?view=meta&token=SEU_TOKEN
```
```json
{
  "lojas": ["PORTO BELO", "IMPERIAL", "MEGA CENTER"],
  "anos": ["2026", "2025"],
  "blocos": [{ "chave": "vendas", "titulo": "Vendas diárias" }, ...],
  "meses": ["JAN", "FEV", ..., "DEZ"]
}
```

### `?view=tidy` — todos os dados, célula a célula
A cópia completa do GD. Cada registro é uma célula preenchida.
Filtros opcionais: `&loja=`, `&ano=`, `&bloco=`. Como é grande, **puxe por loja**.

```
GET /exec?view=tidy&loja=IMPERIAL&ano=2026&token=SEU_TOKEN
```
```json
{
  "geradoEm": "2026-08-29T12:00:00.000Z",
  "total": 1234,
  "registros": [
    {
      "loja": "IMPERIAL",
      "ano": "2026",
      "bloco": "vendas",
      "blocoTitulo": "Vendas diárias",
      "linhaIndice": 33,
      "linha": "TOTAL GERAL",
      "colIndice": 4,
      "coluna": "MAR",
      "celula": "D35",
      "valor": 354156.42
    }
  ]
}
```

Campos do registro:

| campo | descrição |
|---|---|
| `loja`, `ano` | a que loja/ano pertence |
| `bloco`, `blocoTitulo` | seção do GD (chave + nome) |
| `linhaIndice` | posição da linha dentro do bloco (0 = cabeçalho) |
| `linha` | rótulo da linha (ex.: "ALUGUEL", "TOTAL GERAL", ou o dia "15") |
| `colIndice` | número real da coluna na planilha (B = 2) |
| `coluna` | mês (`JAN`..`DEZ`) ou `COL_x` para colunas fora dos meses |
| `celula` | endereço exato na planilha (ex.: `D35`) — use este para gravar de volta |
| `valor` | número, texto ou data |

> Só vêm células **com valor** — células vazias não viram registro.

### `?view=resumo` — KPIs prontos
Indicadores de venda por loja/ano, úteis para dashboards.

```
GET /exec?view=resumo&loja=PORTO BELO&token=SEU_TOKEN
```
```json
{
  "geradoEm": "...",
  "meses": ["JAN", ...],
  "lojas": {
    "PORTO BELO": {
      "2026": {
        "vendasPorMes": [549113.42, ...],
        "vendasTotalAno": 5061034.06,
        "reposicaoPorMes": [310822.56, ...],
        "margemPorMes": [0.7666, ...],
        "vendasAnoAnterior": [536661.59, ...],
        "diferencaValorMes": [12451.83, ...],
        "diferencaPctMes": [0.0232, ...]
      },
      "2025": { ... }
    }
  }
}
```
Observações: `margemPorMes` vem como fração (0.76 = 76%). Meses futuros do ano em curso vêm `0`.

### `?view=raw` — grade fiel por bloco
Devolve os blocos em matriz (linhas × colunas), fiel à planilha. Útil para reproduzir o visual.
Filtros: `&loja=`, `&ano=`, `&bloco=`.

```json
{ "lojas": { "IMPERIAL": { "2026": {
  "vendas": { "titulo": "Vendas diárias", "linhas": [ ["VENDAS", ...], ... ] }
}}}}
```

---

## Endpoint de escrita (POST) — grava no GD

Grava valores nas células do GD da loja. A planilha consolidada reflete sozinha
(via IMPORTRANGE). **Grave apenas campos que o sistema é dono**, para não conflitar com
edição manual.

```
POST /exec
Content-Type: application/json
```
```json
{
  "token": "SEU_TOKEN",
  "escritas": [
    { "loja": "IMPERIAL", "ano": "2026", "celula": "D35", "valor": 354156.42 },
    { "loja": "IMPERIAL", "ano": "2026", "bloco": "vendas", "linha": "15", "coluna": "MAR", "valor": 12767.98 }
  ]
}
```

Duas formas de endereçar cada escrita:
- **Por célula (recomendado):** `celula` no formato A1 (ex.: `D35`) — pegue do `tidy`.
- **Por significado:** `bloco` + `linha` (rótulo da coluna A) + `coluna` (mês, letra ou número).

Resposta:
```json
{
  "recebidos": 2,
  "gravados": 2,
  "resultado": [
    { "i": 0, "ok": true, "loja": "IMPERIAL", "ano": "2026", "celula": "D35" },
    { "i": 1, "ok": false, "erro": "linha não encontrada: 15" }
  ]
}
```
Cada item traz `ok:true/false`; em erro, `erro` explica o motivo daquela escrita.

> Chame o POST **pelo servidor** (backend/edge function). Do navegador, o `Content-Type:
> application/json` dispara *preflight* CORS e costuma falhar.

---

## Erros comuns

| resposta | causa |
|---|---|
| `{ "erro": "token inválido ou ausente" }` | token errado ou faltando |
| `{ "erro": "JSON inválido" }` | corpo do POST não é JSON válido |
| `resultado[i].erro: "loja desconhecida"` | nome da loja fora de PORTO BELO / IMPERIAL / MEGA CENTER |
| `resultado[i].erro: "bloco não encontrado"` | chave de bloco inválida |
| `resultado[i].erro: "linha não encontrada"` | rótulo de linha não bate; use `celula` |
| A URL `.../macros/echo?...` não responde a parâmetros novos | use a URL `/exec` da implantação, não a de redirecionamento |

---

## Exemplos rápidos

Leitura no navegador (cole a URL):
```
/exec?view=resumo&token=SEU_TOKEN
/exec?view=tidy&loja=MEGA CENTER&ano=2026&token=SEU_TOKEN
/exec?view=meta&token=SEU_TOKEN
```

Escrita (por linha de comando):
```bash
curl -L -X POST "https://SEU-APP/exec" \
  -H "Content-Type: application/json" \
  -d '{"token":"SEU_TOKEN","escritas":[{"loja":"IMPERIAL","ano":"2026","celula":"D35","valor":354156.42}]}'
```

---

## Versionamento / atualização

Ao alterar o código do Apps Script, a URL `/exec` só passa a servir o novo código depois de
**reimplantar uma nova versão** da mesma implantação (Implantar ▸ Gerenciar implantações ▸ editar
▸ Versão: Nova versão ▸ Implantar). A URL permanece a mesma.
