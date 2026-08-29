# Dashboard Ricardo's

Dashboard financeiro completo para as lojas Ricardo's — Confresa (Mega Center), Gurupi (Imperial) e Porto (Porto Belo).

Arquivo HTML unico que puxa dados em tempo real da API do GD (Google Apps Script).

---

## Setup Rapido

### Opcao 1 — Comando unico (baixa direto na area de trabalho)

```bash
curl -sL https://raw.githubusercontent.com/samwelltech/dashboard-ricardos/main/dashboard.html -o ~/Desktop/dashboard-ricardos.html && open ~/Desktop/dashboard-ricardos.html
```

### Opcao 2 — Clonar o repositorio

```bash
git clone https://github.com/samwelltech/dashboard-ricardos.git
open dashboard-ricardos/dashboard.html
```

---

## Configuracao

1. Abra o arquivo `dashboard.html` no navegador
2. Na primeira vez, um modal vai pedir:
   - **URL da API** — a URL da implantacao do Google Apps Script (termina em `/exec`)
   - **Token** — o token de autenticacao da API
3. Clique em **Conectar**
4. Os dados ficam salvos no navegador (localStorage), nao precisa preencher de novo

Para alterar a configuracao depois, clique em **Configuracoes** no menu lateral.

---

## Funcionalidades

| Secao | O que mostra |
|---|---|
| **Dashboard** | KPIs de vendas, margem, reposicao. Graficos comparativos ano atual vs anterior. Cards por loja. |
| **Vendas** | Tabela comparativa entre lojas, grafico empilhado, participacao por loja (pizza). |
| **Financeiro** | Entradas vs Saidas, A Receber vs A Pagar. |
| **Despesas** | Despesas vs Impostos, ranking das maiores categorias de gasto. |
| **Estoque** | Compras mensais e evolucao do estoque. |

### Filtros

- **Loja** — Todas, Confresa, Gurupi ou Porto
- **Ano** — 2026 ou 2025
- **Mes** — Mes atual, Ano Todo, ou qualquer mes especifico

---

## Requisitos

- Navegador moderno (Chrome, Edge, Firefox, Safari)
- Acesso a internet (para carregar Chart.js e fontes, e para acessar a API)
- URL e Token da API do GD

---

## Estrutura

```
dashboard.html   ← arquivo unico do dashboard (abra no navegador)
.env.example     ← modelo das credenciais necessarias
API_DOCS.md      ← documentacao completa da API
```

---

## Seguranca

O token da API fica salvo apenas no `localStorage` do navegador onde voce configurou. Nenhuma credencial e incluida no codigo-fonte. O `.env` com credenciais reais esta no `.gitignore`.
