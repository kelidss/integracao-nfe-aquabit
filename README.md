# Módulo de Integração NFe para o Sistema Aquabit

![Status](https://img.shields.io/badge/status-em%20desenvolvimento-blue)
![Python Version](https://img.shields.io/badge/python-3.9+-blue?logo=python&logoColor=white)
![Django Version](https://img.shields.io/badge/django-4.x-green?logo=django&logoColor=white)
![License](https://img.shields.io/badge/license-MIT-green)

> **Resumo:** Serviço de API para automação e gerenciamento da emissão de Notas Fiscais Eletrônicas (NFe) para o sistema Aquabit, com suporte a múltiplos provedores.

---

## 📖 Tabela de Conteúdos

* [Sobre o Projeto](#-sobre-o-projeto)
* [🚀 Principais Funcionalidades](#-principais-funcionalidades)
* [🏛️ Arquitetura da Solução](#️-arquitetura-da-solução)
* [🛠️ Tecnologias Utilizadas](#️-tecnologias-utilizadas)
* [⚙️ Instalação e Configuração](#️-instalação-e-configuração)
* [🔌 Endpoints da API](#-endpoints-da-api)
* [🤝 Como Contribuir](#-como-contribuir)
* [📄 Licença](#-licença)

---

## 📝 Sobre o Projeto

Este projeto implementa a integração com plataformas de emissão de Nota Fiscal Eletrônica (NFe) no sistema **Aquabit**. Seu principal objetivo é automatizar o processo de emissão fiscal para as vendas realizadas na plataforma, oferecendo uma solução robusta, escalável e de fácil manutenção.

A arquitetura foi projetada para ser flexível, permitindo a integração com diferentes provedores de NFe (como Nuvem Fiscal, Focus NFe, etc.) através de um padrão de *Adapter*, centralizando a lógica de negócio e isolando as particularidades de cada API externa.

---

## 🚀 Principais Funcionalidades

* **Emissão Automática de NFe:** Gera notas fiscais de forma assíncrona para as vendas concluídas.
* **Múltiplos Provedores:** Suporte integrado a diferentes APIs de emissão de NFe, configurável por cliente.
* **Gestão Completa:** Permite consultar status, listar notas, e realizar o download de DANFE (PDF) e XML.
* **Histórico e Logs:** Mantém um registro detalhado de todas as transações fiscais para auditoria e depuração.
* **Processamento Assíncrono:** Utiliza Celery para processar as emissões em fila, garantindo que a performance da aplicação principal não seja afetada.

---

## 🏛️ Arquitetura da Solução

A solução é dividida em três camadas principais: **Models** (estrutura de dados), **Services** (lógica de negócio) e **Adapters** (comunicação com as APIs externas).

![Fluxograma Lógico da Emissão](https://mermaid.ink/img/pako:eNplk89uwjAQxl_F8xStAyoQkKoqVUJVetAFK27hSAybBHYqlYrw7p1QqbRSfMv-f2Z-zFtaWJvTAiE1O-3mBQq-8K11gq8a4kS03cTjI1H2G35F-C63uB-bJ0N4-u1xM1d-wJ1n5R-76q0cR9Qp_0B27qS3X08Vz9q1g9Lwz_1T8gQ-3r5m18v6cGYsW1eK2jV1g6L3rN3Vz7lX6sB_t-6f1OexY8u-rQe-rC2s_x6Wn_P36C_Y90xM5N44tY_b84N14WjWf-6kI4vV955H805d2iU8f8N6rM6k-2T6YjT4K1qWJ6c4b2w2x-2XqYn9m-T8tqDdpT6e5j6RhtjV_rYk5Q2W1l1l-j6uF2sL7X5QyFv01h3tV3V121Uf7Fw-9l1p11B8VlJ-tI0kYfF9r_W4p4x0L36n5d0lJmO381oD24_dK2M1-c6oN8n4G8t-X0QzFf1D_M7u-9f8R_jH0Pq_9Q9W1d-fM)

---

## 🛠️ Tecnologias Utilizadas

* **Backend:** Python 3.9+, Django, Django REST Framework
* **Processamento Assíncrono:** Celery
* **Filas (Broker):** Redis / RabbitMQ
* **Banco de Dados:** PostgreSQL

---

## ⚙️ Instalação e Configuração

Siga os passos abaixo para configurar o ambiente de desenvolvimento.

**Pré-requisitos:**
* Python 3.9+
* Git
* Docker (recomendado, para Redis/Postgres)

**Passos:**

1.  **Clone o repositório:**
    ```bash
    git clone [https://github.com/seu-usuario/integracao-nfe-aquabit.git](https://github.com/seu-usuario/integracao-nfe-aquabit.git)
    cd integracao-nfe-aquabit
    ```

2.  **Crie e ative um ambiente virtual:**
    ```bash
    python -m venv venv
    source venv/bin/activate  # No Windows: venv\Scripts\activate
    ```

3.  **Instale as dependências:**
    ```bash
    pip install -r requirements.txt
    ```

4.  **Configure as variáveis de ambiente:**
    * Renomeie o arquivo `.env.example` para `.env`.
    * Preencha as variáveis necessárias, como credenciais do banco de dados e URLs das APIs.
    ```ini
    # .env
    SECRET_KEY=sua-chave-secreta
    DEBUG=True
    DATABASE_URL=postgres://user:password@host:port/dbname

    # Configuração NFe
    NFE_DEFAULT_PROVIDER=nuvem-fiscal
    NFE_TIMEOUT=30
    ```

5.  **Execute as migrações do banco de dados:**
    ```bash
    python manage.py migrate
    ```

6.  **Inicie o servidor de desenvolvimento:**
    ```bash
    python manage.py runserver
    ```

7.  **Inicie o worker do Celery (em um novo terminal):**
    ```bash
    celery -A seuprojeto worker -l info -Q nfe
    ```

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

## 🤝 Como Contribuir

Contribuições são o que tornam a comunidade de código aberto um lugar incrível para aprender, inspirar e criar. Qualquer contribuição que você fizer será **muito apreciada**.

1.  Faça um **Fork** do projeto.
2.  Crie uma **Branch** para sua Feature (`git checkout -b feature/AmazingFeature`).
3.  Faça o **Commit** de suas mudanças (`git commit -m 'Add some AmazingFeature'`).
4.  Faça o **Push** para a Branch (`git push origin feature/AmazingFeature`).
5.  Abra um **Pull Request**.

---

## 📄 Licença

Distribuído sob a licença MIT. Veja `LICENSE.txt` para mais informações.