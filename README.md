# ⚡ CheckMultas · Ferramentas

Conjunto de utilitários web para processamento de **placas veiculares**, infrações e dados do sistema CheckMultas. Todas as ferramentas funcionam **100% no navegador** — sem servidor, sem instalação, sem dependências externas além do SheetJS (carregado via CDN).

---

## 🚀 Como usar

Basta abrir o arquivo `checkmultas-tools.html` diretamente no navegador:

```bash
# Clone o repositório
git clone https://github.com/seu-usuario/checkmultas-tools.git

# Abra o arquivo no navegador
open checkmultas-tools.html
# ou simplesmente dê duplo clique no arquivo
```

Nenhum `npm install`, `build` ou servidor local é necessário.

---

## 🛠️ Ferramentas incluídas

### 🔄 Conversor de Placas — Mercosul ↔ Tradicional

Converte placas entre o formato antigo (`ABC-1234`) e o formato Mercosul (`ABC1D23`) utilizando a tabela oficial do Denatran.

**Funcionalidades:**
- Detecção automática do formato de entrada
- Conversão em lote (cole várias placas de uma vez)
- Suporta entrada suja: `('HKT0416','RGA2D75','RGA3A02')`, separadas por vírgula, espaço ou linha
- Exportação SQL `IN (...)` para originais, convertidas ou ambas
- Cópia para área de transferência com um clique

**Tabela de conversão Denatran (posição 5):**

| Dígito | Letra Mercosul |
|--------|---------------|
| 0      | A             |
| 1      | B             |
| 2      | C             |
| 3      | D             |
| 4      | E             |
| 5      | F             |
| 6      | G             |
| 7      | H             |
| 8      | I             |
| 9      | J             |

---

### ✅ Validador de Placas em Lote

Valida lotes de placas nos dois formatos com diagnóstico detalhado por posição.

**Funcionalidades:**
- Suporta formato tradicional (`AAA0000`) e Mercosul (`AAA0A00`)
- Diagnóstico por posição: indica exatamente qual caractere está errado e o que era esperado
- Separa visualmente placas válidas e inválidas
- Preserva o número da linha original para rastreabilidade
- Exporta válidas e inválidas em arquivos `.xlsx` separados

---

### 📋 Conversor JSON → CSV

Converte um ou múltiplos objetos JSON em CSV compatível com Excel.

**Funcionalidades:**
- Suporta três formatos de entrada: objeto único, array `[{...}]` ou múltiplos objetos colados sequencialmente
- CSV gerado com BOM UTF-8 para abrir corretamente no Excel
- Delimitador ponto e vírgula (`;`) no padrão brasileiro
- Preview dos dados em tabela antes do download
- Extração automática das placas (`placa` / `Placa` / `PLACA`) com geração de SQL `IN (...)`

---

### 📊 Processador de Excel

Lê planilhas `.xlsx` e reorganiza as colunas para o formato padrão do sistema.

**Mapeamento de colunas gerado:**

| Coluna | Campo                   | Observação               |
|--------|-------------------------|--------------------------|
| A      | `placa_Veiculo`         |                          |
| B      | `numero_chassis_veiculo`|                          |
| C      | `numero_renavam`        |                          |
| D      | `codigo_empresa`        |                          |
| E      | `sigla_UF`              |                          |
| F      | `Situação`              |                          |
| G      | `Tipo consulta`         |                          |
| H      | `Tipo Movimentacao`     | Preenchido com `Inclusao`|
| I      | `dt_situacao`           | Converte serial → DD/MM/AAAA |
| J      | `cnpj`                  | Deixado em branco        |
| K      | `UF_venda`              |                          |

**Funcionalidades:**
- Upload por clique ou arrastar e soltar (drag-and-drop)
- Conversão automática de datas em formato serial Excel para `DD/MM/AAAA`
- Preview das primeiras 10 linhas após reorganização
- Exportação CSV com delimitador `;` e encoding UTF-8

---

## 🎨 Interface

- Tema **dark/light** com alternância manual e persistência via `localStorage`
- Detecção automática da preferência do sistema operacional (`prefers-color-scheme`)
- Página inicial com cards de navegação — cada ferramenta abre na mesma página sem recarregar
- Layout responsivo para desktop e mobile

---

## 📦 Dependências

| Biblioteca | Versão  | Uso                                      | Carregamento |
|------------|---------|------------------------------------------|--------------|
| [SheetJS](https://sheetjs.com/) | 0.18.5 | Leitura e escrita de arquivos `.xlsx` | CDN (cdnjs) |

Nenhuma outra dependência. HTML, CSS e JavaScript puros.

---

## 🗂️ Estrutura do projeto

```
checkmultas-tools/
└── checkmultas-tools.html   # Aplicação completa em arquivo único
```

Todo o CSS e JavaScript está inline no HTML para facilitar a distribuição — basta enviar o arquivo.

---

## 🔧 Contexto técnico

Estas ferramentas foram desenvolvidas para suportar operações internas com dados veiculares no padrão brasileiro:

- **Placas** nos formatos Denatran/tradicional e Mercosul
- **Dados** no padrão dos sistemas Denatran / RENAVAM
- **Exportação SQL** compatível com PostgreSQL (`IN (...)` com escape de aspas simples)
- **CSV** compatível com o formato de importação do sistema CheckMultas

---

## 📝 Licença

Uso interno. Sem licença de distribuição pública.
