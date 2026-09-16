# 🚀 Docker CI/CD Multi-Service Validation Pipeline

![Status da Pipeline](https://github.com/MdSFelipe/Docker_in_pipeline/workflows/Docker_in_pipiline/badge.svg)
> **Projeto Prático de Automação e DevOps:** Implementação de uma pipeline de Integração Contínua (CI) que orquestra, valida e analisa o ciclo de vida de múltiplos serviços rodando em containers Docker.

---

## 📌 Visão Geral do Projeto

Este repositório contém uma pipeline automatizada via **GitHub Actions** desenvolvida para demonstrar a manipulação avançada de containers Docker em um ambiente de integração contínua. 

A pipeline executa o ciclo de vida completo para cada serviço:
1. **Download:** Obtenção de imagens oficiais diretamente do Docker Hub.
2. **Inicialização:** Criação e execução de containers com configurações de ambiente e mapeamento de portas.
3. **Execução de Operações:** Execução de scripts, comandos administrativos e varreduras de segurança contra ou dentro dos containers.
4. **Validação:** Verificação automatizada dos resultados de cada operação.
5. **Evidência:** Coleta e exibição dos logs dos containers no console do runner.
6. **Encerramento:** Parada e remoção completa dos containers ao final da execução.

---

## 🛠️ Tecnologias & Imagens Utilizadas

Todas as imagens utilizadas nesta pipeline foram obtidas do registro público oficial **[Docker Hub](https://hub.docker.com)**.

| Serviço | Imagem Docker | Finalidade / Papel no Ecossistema | Container Criado |
| :--- | :--- | :--- | :--- |
| **Python** | `python:3.11-alpine` | Execução de rotinas e scripts de validação lógica | `container-python` |
| **Redis** | `redis:alpine` | Armazenamento de dados em memória e manipulação de chaves | `container-redis` |
| **PostgreSQL** | `postgres:15-alpine` | Banco de dados relacional para execução de consultas SQL | `my-postgres` |
| **Nginx** | `nginx:alpine` | Servidor Web HTTP leve e de alta performance | `container-nginx` |
| **OWASP ZAP** | `zaproxy/zap-stable` | Ferramenta de testes dinâmicos de segurança (DAST) | `container-zap` |

---

## 💻 Comandos Docker Empregados

A pipeline utiliza a linha de comando do Docker nativa para gerenciar todo o ciclo de vida dos containers:

* **`docker run`**: Inicializa os containers a partir das imagens selecionadas.
  * `-d` (*detached*): Mantém o serviço em execução em segundo plano.
  * `-p` (*port mapping*): Expõe as portas internas do container para o host de CI/CD.
  * `-e` (*environment*): Injeta variáveis de ambiente cruciais (como credenciais do banco).
  * `--net=host`: Compartilha a pilha de rede do host para conectar o scanner de segurança ao servidor web.
* **`docker exec`**: Interage com containers em execução para aplicar comandos internos (`redis-cli`, `pg_isready`, `psql`).
* **`docker logs`**: Captura a saída de terminal de cada container para fins de auditoria e relatórios nos logs da pipeline.
* **`docker stop` / `docker rm`**: Garante a limpeza do ambiente de execução e encerramento dos recursos alocados.

---

## ⚙️ Como a Pipeline Valida os Serviços

A validação ocorre de forma automatizada através de checagens assertivas em cada *job*:

1. **Python (`container-python`):** Executa uma asserção de código `python -c "assert 1 + 1 == 2"`. A validação é confirmada pelo código de saída `0` do container.
2. **Redis (`container-redis`):** Escreve a chave `status` com o valor `"Redis Operational"` via `redis-cli` e valida a persistência em memória aplicando um `grep` no comando de leitura.
3. **PostgreSQL (`my-postgres`):** Monitora a prontidão do serviço usando o utilitário `pg_isready`. Após a confirmação, executa a instrução SQL `SELECT 'Operacao realizada com sucesso!' AS status;` e valida a execução do comando.
4. **Nginx (`container-nginx`):** Envia uma requisição HTTP HEAD via `curl` contra a porta `8080` do host e valida o recebimento do código de status `200 OK`.
5. **OWASP ZAP (`container-zap`):** Executa o scanner DAST `zap-baseline.py` apontado para o container Nginx, auditando o servidor ativo e gerando o relatório de segurança nos logs da execução.

---

## 📊 Evidências de Execução

Todas as evidências de execução, status dos *jobs* e relatórios de logs gerados pelos comandos `docker logs` podem ser acompanhados diretamente na aba **Actions** deste repositório.

Os containers são parados e removidos automaticamente ao término da pipeline através da instrução `if: always()`, garantindo o isolamento do ambiente de execução a cada *commit*.
