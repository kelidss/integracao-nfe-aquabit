# Módulo de Integração NFe para o Sistema Aquabit

![Status](https://img.shields.io/badge/status-em%20desenvolvimento-blue)
![Python Version](https://img.shields.io/badge/python-3.9+-blue?logo=python&logoColor=white)
![Django Version](https://img.shields.io/badge/django-4.x-green?logo=django&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)


## 📝 Sobre o Projeto

Este projeto implementa a integração com plataformas de emissão de Nota Fiscal Eletrônica (NFe) no sistema **Aquabit**. Seu principal objetivo é automatizar o processo de emissão fiscal para as vendas realizadas na plataforma, oferecendo uma solução robusta, escalável e de fácil manutenção.

A arquitetura foi projetada para ser flexível, permitindo a integração com diferentes provedores de NFe (como Nuvem Fiscal, Focus NFe, etc.) através de um padrão de *Adapter*, centralizando a lógica de negócio e isolando as particularidades de cada API externa.

---

## 🔌 Endpoints da API

A API oferece endpoints para gerenciar notas fiscais e suas configurações.

### Notas Fiscais (`/api/v2/nfe/`)

| Método | Endpoint                | Descrição                               |
| :----- | :---------------------- | :-------------------------------------- |
| `GET`  | `/`                       | Lista todas as NFes emitidas.           |
| `POST` | `/{id}/emitir/`           | Dispara o processo de emissão da NFe.   |
| `GET`  | `/{id}/consultar/`        | Consulta o status mais recente da NFe.  |
| `GET`  | `/{id}/baixar_danfe/`     | Realiza o download do PDF da DANFE.     |
| `GET`  | `/{id}/baixar_xml/`       | Realiza o download do arquivo XML.      |

### Configurações (`/api/v2/configuracao-nfe/`)

| Método | Endpoint | Descrição                               |
| :----- | :------- | :-------------------------------------- |
| `GET`  | `/`        | Lista todas as configurações.         |
| `POST` | `/`        | Cria uma nova configuração.           |

---
# 📊 DIAGRAMA NFe - Versão Visual ASCII

## 🎯 RELACIONAMENTOS PRINCIPAIS

```
┌────────────────────────────────────────────────────────────────────────────┐
│                        MÓDULO CORE (EXISTENTE)                             │
└────────────────────────────────────────────────────────────────────────────┘

┌──────────────────┐
│   PROPRIEDADE    │
│ ════════════════ │
│ • uuid           │
│ • nome           │
│ • cnpj           │
│ • inscricao_est  │
│ • cpf_dono       │
│ • endereco       │
│ • estado         │
│ • cidade         │
└──────────────────┘
         │
         ├─────────────────────┐
         │                     │
         ▼                     ▼
┌──────────────────┐  ┌──────────────────┐
│     CLIENTE      │  │  FORNECEDOR      │
│ ════════════════ │  │ ════════════════ │
│ • uuid           │  │ • uuid           │
│ • nome           │  │ • nome           │
│ • cpf_cnpj       │  │ • cpf_cnpj       │
│ • endereco       │  │ • endereco       │
│ • estado         │  │ • estado         │
│ • cidade         │  │ • cidade         │
│ • email          │  │ • email          │
└──────────────────┘  └──────────────────┘
         │
         │
         ▼
┌──────────────────────────────────────┐
│      CATEGORIA FINANCEIRA            │
│ ════════════════════════════════════ │
│ • uuid                               │
│ • propriedade_id                     │
│ • tipo (vendas/despesas/produtos)    │
│ • nome                               │
│ • foto                               │
└──────────────────────────────────────┘
         │
         ▼
┌──────────────────────────────────────┐
│         MOVIMENTAÇÃO                 │
│ ════════════════════════════════════ │
│ • uuid                               │
│ • propriedade_id                     │
│ • cliente_id                         │
│ • categoria_id                       │
│ • data                               │
│ • valor_total                        │
│ • valor_frete                        │
│ • desconto                           │
│ • descricao                          │
└──────────────────────────────────────┘
         │
         ├─── tem vários ───►
         ▼
┌──────────────────────────────────────┐
│       MOVIMENTAÇÃO ITEM              │
│ ════════════════════════════════════ │
│ • uuid                               │
│ • movimentacao_id                    │
│ • categoria_id                       │
│ • descricao                          │
│ • quantidade                         │
│ • valor_unitario                     │
│ • valor_total                        │
└──────────────────────────────────────┘


┌────────────────────────────────────────────────────────────────────────────┐
│                    NOVO MÓDULO INTEGRACAO (NFE)                            │
└────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────┐
│        PROVEDOR NFe                  │  ◄── GLOBAL (não vinculado)
│ ════════════════════════════════════ │
│ • uuid                               │
│ • nome (Revgás, Nuvem Fiscal)        │
│ • slug                               │
│ • api_url                            │
│ • versao_api                         │
│ • suporta_homologacao                │
│ • suporta_cancelamento               │
└──────────────────────────────────────┘
         │
         └─── usado em ───►
                          │
┌─────────────────────────▼──────────────────┐
│   CONFIGURAÇÃO NFe PROPRIEDADE             │  ◄── POR PROPRIEDADE
│ ══════════════════════════════════════════ │
│ • uuid                                     │
│ • propriedade_id (OneToOne)                │
│ • provedor_nfe_id                          │
│ • ambiente (homolog/produção)              │
│ • api_token                                │
│ • certificado_digital                      │
│ • serie_nfe (920)                          │
│ • proximo_numero                           │
│ • natureza_operacao_padrao                 │
│ • regime_tributario (Simples Nacional)     │
└────────────────────────────────────────────┘
         │
         └─── emite ───►
                       │
┌──────────────────────▼─────────────────────┐
│         CATEGORIA NFe                      │  ◄── DADOS FISCAIS
│ ══════════════════════════════════════════ │
│ • uuid                                     │
│ • categoria_financeira_id (OneToOne)       │
│ • ncm (03043900)                           │
│ • cfop (5101)                              │
│ • cst_icms (102)                           │
│ • cst_pis (49)                             │
│ • cst_cofins (49)                          │
│ • unidade_medida (KG)                      │
│ • descricao_fiscal                         │
└────────────────────────────────────────────┘
         │
         └─── classifica ───►
                            │
┌───────────────────────────▼───────────────────────┐
│              NOTA FISCAL                          │
│ ═════════════════════════════════════════════════ │
│ • uuid                                            │
│ • movimentacao_id (OneToOne) ◄──────┐            │
│ • configuracao_nfe_id                │            │
│ • cliente_id                         │            │
│ • numero_nota                        │            │
│ • serie (920)                        │            │
│ • chave_acesso (44 dígitos)          │            │
│ • protocolo_autorizacao              │            │
│ • data_emissao                       │            │
│ • data_autorizacao                   │            │
│ • status (pendente/autorizada)       │            │
│ • valor_total                        │            │
│ • xml_content                        │            │
│ • pdf_danfe                          │            │
└───────────────────────────────────────────────────┘
         │
         ├─── contém ───►
         │              │
         ▼              ▼
┌─────────────────────────────────┐  ┌─────────────────────────────┐
│    NOTA FISCAL ITEM             │  │  FORMA PAGAMENTO NFe        │
│ ═══════════════════════════════ │  │ ═══════════════════════════ │
│ • uuid                          │  │ • uuid                      │
│ • nota_fiscal_id                │  │ • nota_fiscal_id            │
│ • movimentacao_item_id ◄────┐   │  │ • forma_pagamento (01)      │
│ • categoria_nfe_id           │   │  │ • indicador (0=vista)       │
│ • numero_item                │   │  │ • valor_pagamento           │
│ • codigo_produto             │   │  └─────────────────────────────┘
│ • descricao                  │   │
│ • ncm                        │   │  ┌─────────────────────────────┐
│ • cfop                       │   │  │      LOG NFe                │
│ • unidade (KG)               │   │  │ ═══════════════════════════ │
│ • quantidade                 │   │  │ • uuid                      │
│ • valor_unitario             │   │  │ • nota_fiscal_id            │
│ • valor_total                │   │  │ • operacao (emissao)        │
│ • impostos (JSON)            │   │  │ • status_code               │
└─────────────────────────────────┘  │ • requisicao                │
                                     │ • resposta                  │
                                     │ • erro                      │
                                     │ • tempo_resposta            │
                                     └─────────────────────────────┘
```

---

## 🔗 FLUXO DE DADOS PASSO A PASSO

```
PASSO 1: DADOS EXISTENTES
══════════════════════════

┌─────────────┐
│ Movimentação│ ──► tipo: "VENDA"
│   ID: 8089  │     valor_total: R$ 12.000
└─────────────┘     cliente_id: 3421
      │             categoria_id: 150
      │
      ├──► MovimentacaoItem
      │    ├─ ID: 15234
      │    ├─ descricao: "PANGA"
      │    ├─ quantidade: 1500 KG
      │    └─ valor_unitario: R$ 8,00
      │
      └──► Cliente
           ├─ ID: 3421
           ├─ nome: "JORGE"
           └─ cpf_cnpj: "123.456.789-00"


PASSO 2: CONFIGURAÇÕES NFe
═══════════════════════════

ConfiguracaoNFePropriedade
├─ provedor: Revgás
├─ ambiente: homologação
├─ serie: 920
└─ proximo_numero: 38

CategoriaNFe (categoria 150)
├─ ncm: 03043900
├─ cfop: 5101
├─ cst_icms: 102
└─ unidade: KG


PASSO 3: GERAÇÃO DA NFe
════════════════════════

NotaFiscal
├─ numero: 38
├─ serie: 920
├─ chave: 22251100039741095368559200000000381234567893
├─ status: "autorizada"
└─ protocolo: 222250018710133

NotaFiscalItem (baseado em MovimentacaoItem 15234)
├─ numero_item: 1
├─ descricao: "PANGA CONGELADA"
├─ ncm: 03043900
├─ quantidade: 1500.0000 KG
├─ valor_unitario: 8.0000000000
└─ valor_total: 12000.00

FormaPagamentoNFe
├─ forma: "01" (Dinheiro)
├─ indicador: "0" (À vista)
└─ valor: 12000.00
```

---

## 📋 TABELA DE RELACIONAMENTOS

| Tabela Origem | Tipo Relação | Tabela Destino | Cardinalidade |
|---------------|--------------|----------------|---------------|
| **Propriedade** | possui | ConfiguracaoNFePropriedade | 1:1 |
| **Propriedade** | tem | Cliente | 1:N |
| **Propriedade** | tem | CategoriaFinanceira | 1:N |
| **Propriedade** | tem | Movimentacao | 1:N |
| **CategoriaFinanceira** | configura | CategoriaNFe | 1:1 |
| **Movimentacao** | contém | MovimentacaoItem | 1:N |
| **Movimentacao** | referencia | Cliente | N:1 |
| **Movimentacao** | gera | NotaFiscal | 1:0..1 |
| **ProvedorNFe** | usado em | ConfiguracaoNFePropriedade | 1:N |
| **ConfiguracaoNFePropriedade** | emite | NotaFiscal | 1:N |
| **NotaFiscal** | contém | NotaFiscalItem | 1:N |
| **NotaFiscal** | tem | FormaPagamentoNFe | 1:N |
| **NotaFiscal** | registra | LogNFe | 1:N |
| **MovimentacaoItem** | origina | NotaFiscalItem | 1:1 |
| **CategoriaNFe** | classifica | NotaFiscalItem | 1:N |

---

## 🎨 CORES E SÍMBOLOS

```
✅ = EXISTENTE (não precisa criar)
🆕 = NOVO (precisa implementar)
🔗 = RELACIONAMENTO
📊 = DADOS
⚙️ = CONFIGURAÇÃO
📄 = DOCUMENTO GERADO
```

### LEGENDA:

```
┌──────────────┐
│ CORE         │  ✅ Models que já existem no sistema
└──────────────┘

┌──────────────┐
│ INTEGRACAO   │  🆕 Models novos para NFe
└──────────────┘

  ──►  Relacionamento direto (ForeignKey)
  ═══  OneToOne relationship
  ┤├─  ManyToMany (não usado aqui)
```

---

## 📊 EXEMPLO VISUAL COMPLETO

```
EXEMPLO: Venda de 1.500 KG de PANGA por R$ 12.000
════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────────┐
│ 1. MOVIMENTAÇÃO (CORE) ✅                                   │
├─────────────────────────────────────────────────────────────┤
│ ID: 8089                                                    │
│ Propriedade: Fazenda São José (ID: 2656)                   │
│ Data: 03/11/2025 10:30                                      │
│ Cliente: JORGE (ID: 3421)                                   │
│ Categoria: Vendas de Peixe (ID: 150)                        │
│ Valor Total: R$ 12.000,00                                   │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. MOVIMENTAÇÃO ITEM (CORE) ✅                              │
├─────────────────────────────────────────────────────────────┤
│ ID: 15234                                                   │
│ Movimentação ID: 8089                                       │
│ Descrição: PANGA                                            │
│ Quantidade: 1.500,0000 KG                                   │
│ Valor Unitário: R$ 8,00                                     │
│ Valor Total: R$ 12.000,00                                   │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. CONFIGURAÇÃO NFe (INTEGRACAO) 🆕                         │
├─────────────────────────────────────────────────────────────┤
│ Propriedade ID: 2656                                        │
│ Provedor: Revgás                                            │
│ Ambiente: Homologação                                       │
│ Série: 920                                                  │
│ Próximo Número: 38                                          │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. CATEGORIA NFe (INTEGRACAO) 🆕                            │
├─────────────────────────────────────────────────────────────┤
│ Categoria Financeira ID: 150                                │
│ NCM: 03043900 (Peixes congelados)                           │
│ CFOP: 5101 (Venda dentro do estado)                         │
│ CST ICMS: 102 (Simples Nacional)                            │
│ CST PIS: 49 (Outras operações)                              │
│ CST COFINS: 49 (Outras operações)                           │
│ Unidade: KG                                                 │
│ Descrição Fiscal: PANGA CONGELADA                           │
└─────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. NOTA FISCAL (INTEGRACAO) 🆕                              │
├─────────────────────────────────────────────────────────────┤
│ ID: 12                                                      │
│ Número: 38                                                  │
│ Série: 920                                                  │
│ Chave: 22251100039741095368559200000000381234567893         │
│ Status: AUTORIZADA ✅                                        │
│ Protocolo: 222250018710133                                  │
│ Data Emissão: 03/11/2025 10:35:00                           │
│ Data Autorização: 03/11/2025 10:35:15                       │
│ Valor Total: R$ 12.000,00                                   │
│ XML: <nfeProc>...</nfeProc> 📄                              │
│ DANFE: nfe_38_920.pdf 📄                                    │
└─────────────────────────────────────────────────────────────┘
         │
         ├────────────────────┬────────────────────┐
         ▼                    ▼                    ▼
┌──────────────────┐  ┌─────────────────┐  ┌──────────────┐
│ NOTA FISCAL ITEM │  │ FORMA PAGAMENTO │  │   LOG NFe    │
│ 🆕               │  │ 🆕              │  │   🆕         │
├──────────────────┤  ├─────────────────┤  ├──────────────┤
│ ID: 87           │  │ ID: 45          │  │ ID: 234      │
│ Número: 1        │  │ Forma: 01       │  │ Operação:    │
│ Código: 001      │  │ (Dinheiro)      │  │ emissao      │
│ NCM: 03043900    │  │ Indicador: 0    │  │ Status: 200  │
│ CFOP: 5101       │  │ (À vista)       │  │ Tempo: 2.7s  │
│ Qtd: 1500.0000   │  │ Valor: 12000.00 │  │ Sucesso ✅   │
│ Vlr Unit: 8.00   │  └─────────────────┘  └──────────────┘
│ Vlr Total:       │
│ 12000.00         │
└──────────────────┘
```

---

## 💡 RESUMO

### ✅ O QUE JÁ TEMOS (CORE):
1. **Propriedade** - Fazenda com CNPJ
2. **Cliente** - Comprador
3. **CategoriaFinanceira** - Tipo de produto (ex: Vendas de Peixe)
4. **Movimentacao** - Venda registrada
5. **MovimentacaoItem** - Detalhes da venda (1500 KG PANGA)

### 🆕 O QUE PRECISAMOS CRIAR (INTEGRACAO):

1. **ProvedorNFe** (GLOBAL)
   - Cadastro de provedores: Revgás, Nuvem Fiscal, etc
   - Um cadastro serve para todas as propriedades

2. **ConfiguracaoNFePropriedade** (POR PROPRIEDADE)
   - Cada fazenda configura qual provedor usar
   - Guarda: token, certificado, série, próximo número

3. **CategoriaNFe** (DADOS FISCAIS)
   - Para cada categoria financeira, define:
   - NCM, CFOP, CST (códigos fiscais)

4. **NotaFiscal** (DOCUMENTO)
   - Criada a partir da Movimentação
   - Contém: chave 44 dígitos, XML, DANFE PDF

5. **NotaFiscalItem** (ITENS DO DOCUMENTO)
   - Um para cada MovimentacaoItem
   - Com todos os dados fiscais

6. **FormaPagamentoNFe** (OBRIGATÓRIO NFe 4.0)
   - Como foi pago: dinheiro, cartão, etc

7. **LogNFe** (AUDITORIA)
   - Registra todas operações e erros

---

## 🎯 TOTAL DE TABELAS

| Status | Quantidade | Módulo |
|--------|-----------|--------|
| ✅ Existentes | 6 tabelas | core |
| 🆕 Novas | 7 tabelas | integracao |
| **TOTAL** | **13 tabelas** | **2 módulos** |



## 📄 Licença

Distribuído sob a licença MIT. Veja `LICENSE.txt` para mais informações.
