# 📡 Automações Ubiquiti (Relatórios e Monitoramento de Quedas)

---

Este repositório contém dois *workflows* de automação desenvolvidos em **n8n** (nós JSON) para monitoramento e relatórios de dispositivos **Ubiquiti**.

O objetivo principal é garantir a **proatividade** na gestão da infraestrutura de rede, detectando falhas em tempo real e gerando análises mensais robustas sobre a disponibilidade dos dispositivos.

## 🚀 Workflow 1: Relatório Mensal de Dispositivos Offline

Este *workflow* é disparado mensalmente e gera um relatório PDF detalhado sobre as quedas de dispositivos registradas no mês anterior.

### 1. Processamento e Agregação de Dados 📊

O núcleo do *workflow* (`Processar e Estruturar`) é um nó **Code** em JavaScript que transforma dados brutos em inteligência acionável:

* **Coleta:** Puxa todos os registros de quedas (`device_outages`) de uma tabela em **Supabase/PostgreSQL**.
* **Agregação:** Consolida os dados por dispositivo (`device_name`), calculando o **Total de Quedas** e o **Total de Minutos Offline**.
* **Cálculo:** Determina métricas de resumo, como o dispositivo com **Mais Quedas** e o com **Maior Tempo Offline**.

### 2. Geração e Distribuição do Relatório 📧

Após o processamento, o fluxo automatiza a entrega do relatório:

* **HTML/PDF:** O nó **Code** gera um documento HTML estilizado, que é então enviado para a **API PDF.co** (`HTTP Request PDF`) para conversão em um arquivo PDF profissional.
* **Nome Dinâmico:** O nome do arquivo PDF é gerado dinamicamente com o mês e ano de referência (Ex: `Relatorio_Mensal_Dispositivos_Ubiquiti-Novembro-2025.pdf`).
* **Envio:** O nó `Envia E-mail` envia o link para download do PDF para o destinatário, finalizando o ciclo de *reporting*.

---

## 🛠️ Workflow 2: Monitoramento de Queda e Restauração em Tempo Real

Este *workflow* é disparado a cada minuto para verificar o status de todos os dispositivos Ubiquiti e enviar alertas imediatos de queda ou restauração de sinal.

### 1. Detecção de Status e Filtragem 🚨

O sistema monitora a API do Ubiquiti e filtra os estados críticos:

* **Consulta:** O nó `GetUbiquitiDevices` consulta a **API Ubiquiti** para obter o status atual de todos os *hosts*.
* **Filtro:** O nó `FiltraOffline` identifica dispositivos onde `reportedState.state` é igual a **`disconnected`**.

### 2. Gerenciamento do Ciclo de Vida da Queda ✨

O *workflow* gerencia o ciclo completo da queda usando duas tabelas no **Supabase/PostgreSQL**:

* **Dispositivo Offline (Alerta de Queda):**
    * O nó `Puxa Dados Tabela Offline` verifica se o dispositivo já está registrado como offline.
    * Se **NÃO** estiver registrado (`Verifica Se Já Foi Enviado Aviso`), o sistema envia o **Alerta de Queda** (`Envia E-mail Queda`) e cria um registro inicial na tabela (`Cria Memoria de Quedas`) com o `start_time`.
* **Dispositivo Online (Alerta de Restauração):**
    * O nó `Verifica Restauracao` busca registros na tabela de offline.
    * Se um dispositivo que estava offline **agora está online** (`Decide Restauracao`), o sistema:
        1.  **Calcula a Duração** da queda (tempo entre `start_time` e o momento atual).
        2.  Envia o **Alerta de Restauração** (`Envia E-mail Restauracao`).
        3.  **Atualiza** o registro na tabela de relatórios com `end_time` e `duration_minutes`.
        4.  **Remove** o registro da tabela de dispositivos offline.

---

## ⚙️ Configuração Comum

Ambos os *workflows* exigem as seguintes credenciais para funcionar corretamente:

### Requisitos
* Instância ativa do **n8n**.
* Credenciais de API para:
    * **Ubiquiti API Key** (para o `GetUbiquitiDevices`).
    * **Supabase/PostgreSQL** (para as tabelas de `device_outages` e `dispositivos_offline`).
    * **PDF.co API Key** (Apenas para o Relatório Mensal).
    * **SMTP** (para envio de e-mails de alerta e relatório).

> **⚠️ Observação:** O workflow de monitoramento (`Ubiquiti_New`) deve ser configurado com o `Schedule Trigger` para executar a **cada minuto** para detecção em tempo real.
