# Especificação de Alterações (Patch Notes) - REVISÃO FINAL ESTRITA

## 1. Contexto de Execução e Regras Críticas
- **Arquivos Alvo:** `index.html` (Transporte) e `combustiveis.html`. E criação por clonagem de `emprego.html`.
- **Stack:** HTML5 Semântico, CSS3 embutido (tag `<style>`), Vanilla JS embutido (tag `<script>`).
- **REGRA ABSOLUTA DE AMBIENTE:** É estritamente proibido criar arquivos `.js` externos, scripts Node.js, ou ferramentas de automação. Todo o código manipulado e gerado deve residir integralmente dentro dos arquivos `.html` supracitados.
- **REGRA DE RESPONSIVIDADE:** A responsividade original do grid de cards (quebra para 2 e 1 coluna) deve ser preservada intacta. Os novos elementos (Sidebar e Modal Expandido) devem obrigatoriamente possuir regras de `@media (max-width: 768px)` para evitar quebra horizontal em dispositivos móveis.

## 2. Refatoração de Layout: Menu Lateral (Navegação)
- **Alvo:** Estrutura global do `<body>` e cabeçalho de ambos os arquivos.
- **Ações:**
  - Remover o botão/link superior direito (alternância de páginas).
  - Envolver todo o conteúdo principal (abaixo do cabeçalho) em um `<div class="app-layout">`.
  - Aplicar no CSS: `.app-layout { display: grid; grid-template-columns: 250px 1fr; min-height: 100vh; max-width: 1400px; margin: 0 auto; }`
  - Na coluna esquerda, injetar um `<aside class="sidebar">`.
  - O `<aside>` deve conter um título e uma `<nav class="sidebar-nav">` com links (`<a>`) para os arquivos físicos: "Transporte" (`index.html`), "Combustíveis" (`combustiveis.html`) e "Emprego" (`emprego.html`).
  - O link correspondente à página atual deve receber a classe `.active`.
  - A coluna direita (`<main class="container">`) abrigará o grid de cards.
- **Adaptação Responsiva:** Adicionar no CSS: `@media (max-width: 768px) { .app-layout { grid-template-columns: 1fr; } .sidebar { border-right: none; border-bottom: 1px solid var(--border-color); } }`

## 3. Armazenamento de Dados do Dicionário (Local por Card)
- **Alvo:** Estrutura de todos os `<article class="card">` existentes.
- **Ação:** 
  - O dicionário não pode ser estático no modal. Injetar dentro de CADA card, logo após a `<div class="card-body">`, um container oculto com a tabela base do dicionário.
  - Estrutura exigida dentro de cada card:
    ```html
    <div class="card-dictionary-data" style="display: none;">
        <p>O dicionário de dados detalha a estrutura, campos e restrições do dataset referente a esta base.</p>
        <table style="width:100%; border-collapse: collapse; margin-top: 15px;">
            <thead>
                <tr style="border-bottom: 1px solid #ccc; text-align: left;">
                    <th>Campo</th>
                    <th>Tipo</th>
                    <th>Descrição</th>
                </tr>
            </thead>
            <tbody>
                <tr style="border-bottom: 1px solid #eee;">
                    <td>id</td>
                    <td>INT</td>
                    <td>Identificador único do registro</td>
                </tr>
                <tr>
                    <td>valor</td>
                    <td>FLOAT</td>
                    <td>Valor aferido</td>
                </tr>
            </tbody>
        </table>
    </div>
    ```

## 4. Refatoração do Modal (Pop-up): Botões Funcionais
- **Alvo:** Container de botões dentro do modal de download atual (`#downloadModal`).
- **Ações:**
  - **Botão 1 (CSV):** Refatorar para `<a href="#" class="btn-modal btn-primary" download><i class="fa-solid fa-file-csv"></i> Baixar CSV</a>`.
  - **Botão 2 (BigQuery):** Refatorar para `<a href="#" class="btn-modal btn-secondary" target="_blank" rel="noopener noreferrer"><i class="fa-solid fa-code"></i> Consultar BigQuery</a>`.
  - **Novo Botão (Dicionário):** Inserir `<button class="btn-modal btn-dictionary"><i class="fa-solid fa-book"></i> Dicionário de Dados</button>` com as mesmas propriedades estruturais para manter o alinhamento (flexbox com `gap`). Adicionar CSS para `.btn-dictionary` (fundo transparente, borda de contorno).

## 5. Máquina de Estados e Injeção Dinâmica no Modal
- **Regras:** É terminantemente proibida a criação de um segundo `<dialog>` ou `.modal-overlay`.
- **Estrutura do Modal:**
  - O interior da `.modal-body` deve ser subdividido em dois containers:
    1. `<div class="modal-view-actions">` (contendo os botões da etapa 4).
    2. `<div class="modal-view-dictionary" style="display: none;">` (contendo botão `<button class="btn-modal btn-back-dict"><i class="fa-solid fa-arrow-left"></i> Voltar</button>`, título `<h3 class="dict-title">Dicionário de Dados</h3>` e a área alvo `<div class="dict-content" id="dynamicDictContent"></div>`).
- **Comportamento (Vanilla JS na tag `<script>`):**
  - **Injeção de Dados:** Ao clicar no botão de Download de um card, capturar o `innerHTML` do `.card-dictionary-data` (daquele card específico) e injetar em `#dynamicDictContent`.
  - **Expandir Modal:** Ao clicar em `.btn-dictionary`, o JS adiciona a classe `.expanded` em `.modal-content`. Oculta `.modal-view-actions` (`display: none`) e exibe `.modal-view-dictionary` (`display: block`).
  - **Retrair Modal:** Ao clicar no botão Voltar (`.btn-back-dict`), o JS remove a classe `.expanded`, exibe as ações e oculta o dicionário. Ao fechar o modal completamente (botão X ou ESC), o estado também deve ser resetado para a visualização inicial.
- **Estilo de Expansão (CSS):**
  - A classe `.expanded` no `.modal-content` deve transicionar para `width: 70vw; max-width: 900px; max-height: 80vh;`.
  - O container `.modal-view-dictionary` deve possuir `overflow-y: auto; max-height: calc(80vh - 65px);`.
  - **Adaptação Responsiva do Modal:** Adicionar no CSS: `@media (max-width: 768px) { .modal-content.expanded { width: 95vw; max-height: 90vh; } }`