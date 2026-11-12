# Especificação de Requisitos de Software (SRS)

**Projeto:** Sistema de Agendamento para Barbearia  
**Versão:** 1.0 (MVP)  


---

## Sumário
- [1. Visão Geral](#1-visão-geral)
- [2. Requisitos Funcionais (RF)](#2-requisitos-funcionais-rf)
- [3. Regras de Negócio (RN)](#3-regras-de-negócio-rn)
- [4. Requisitos Não Funcionais (RNF)](#4-requisitos-não-funcionais-rnf)
- [5. Modelagem de Dados](#5-modelagem-de-dados)

---

## 1. Visão Geral
Esta aplicação web tem como objetivo modernizar o agendamento online de serviços de barbearia e centralizar o controle diário da agenda dos profissionais.

- **Cliente:** Usuário externo que consulta serviços, verifica horários disponíveis e realiza ou cancela agendamentos sem necessidade de criar conta.
- **Barbeiro (Administrador):** Profissional autenticado que gerencia sua agenda, visualiza os horários gerais da barbearia, lança encaixes manuais, bloqueia horários e administra o catálogo de serviços.

---

## 2. Requisitos Funcionais (RF)

### 2.1 Módulo do Cliente (Público)

**Fluxo Sequencial de Agendamento:**  
`1. Seleção de Serviços (cálculo de duração)` -> `2. Escolha do Barbeiro` -> `3. Seleção de Data e Horário Livre` -> `4. Identificação (Nome/WhatsApp) e Confirmação`

- **RF01 — Catálogo de Serviços:** Exibir serviços ativos com nome, descrição, valor e duração em minutos.
- **RF02 — Seleção Múltipla de Serviços:** Permitir selecionar um ou mais serviços na mesma reserva, somando automaticamente o valor total e a duração acumulada (ex: *Corte 30 min + Barba 15 min = 45 min*).
- **RF03 — Escolha de Profissional:** Permitir escolher um barbeiro específico ou a opção *"Sem preferência"*.
- **RF04 — Cálculo de Horários Livres:** Exibir dinamicamente apenas os horários em que o profissional possui tempo contínuo livre suficiente para realizar todos os serviços selecionados.
- **RF05 — Agendamento Simplificado:** Concluir a reserva informando apenas **Nome Completo** e **WhatsApp**.
- **RF06 — Comprovante e Rastreio:** Gerar um código único de confirmação (ex: `BRB-4829`) e exibir botão para enviar detalhes ou tirar dúvidas via link direto do WhatsApp (`wa.me`).
- **RF07 — Consulta e Cancelamento:** Permitir consultar e cancelar o agendamento informando o código único, respeitando a antecedência mínima exigida.

### 2.2 Módulo do Barbeiro (Autenticado)
- **RF08 — Login Seguro:** Autenticação via e-mail e senha utilizando token JWT.
- **RF09 — Agenda Individual e Geral:** Visualizar os agendamentos por data, permitindo filtrar por um barbeiro específico ou visualizar a agenda completa da barbearia.
- **RF10 — Atualização de Status:** Marcar atendimentos como **Concluído** ou **Cancelado**.
- **RF11 — Encaixe Manual e Bloqueios:** Registrar agendamentos presenciais ou via WhatsApp diretamente no painel e bloquear intervalos de horário (almoço, folgas).
- **RF12 — CRUD de Serviços e Profissionais:** Cadastrar, editar e ativar/desativar serviços e barbeiros.
- **RF13 — Resumo Operacional:** Exibir total de atendimentos concluídos e faturamento estimado (filtrado por dia ou mês).

---

## 3. Regras de Negócio (RN)

- **RN01 — Horário de Funcionamento:** A barbearia funciona de **terça-feira a sábado**, das **09:00 às 19:00**, para todos os profissionais. Nenhum agendamento pode iniciar antes das 09:00 ou terminar após as 19:00.
- **RN02 — Granularidade e Validação de Janela Contínua:** A grade de horários base opera em intervalos de **15 minutos** (ex: 09:00, 09:15, 09:30). Um horário inicial só é exibido como disponível se todo o intervalo contínuo necessário (`start_time` + duração total dos serviços) estiver livre de sobreposições com agendamentos ativos ou bloqueios, e não ultrapassar o horário de fechamento (19:00).
- **RN03 — Janela de Agendamento:**
  - **Antecedência mínima:** 30 minutos em relação ao horário atual.
  - **Limite futuro:** Até 15 dias corridos a partir da data atual.
- **RN04 — Opção "Sem Preferência":** Exibe todos os horários em que ao menos um barbeiro esteja disponível para a duração total solicitada. Na confirmação, o sistema atribui automaticamente o agendamento ao barbeiro com **menor volume de minutos agendados naquele dia**.
- **RN05 — Política de Cancelamento:** O cliente pode cancelar pelo site até **2 horas antes** do horário marcado. Após esse limite, o cancelamento online é bloqueado e exibe um link direto para o WhatsApp da barbearia.
- **RN06 — Prevenção de Conflitos:** Nenhum agendamento ativo (`CONFIRMED` ou `BLOCKED`) pode sobrepor o intervalo de outro registro ativo para o mesmo barbeiro.
- **RN07 — Encaixe Manual:** O agendamento inserido pelo painel administrativo ignora a antecedência mínima de 30 minutos (RN03), mas continua respeitando a disponibilidade do horário livre (RN06).

---

## 4. Requisitos Não Funcionais (RNF)

- **RNF01 — Responsividade (Mobile-First):** Layout adaptado prioritariamente para dispositivos móveis.
- **RNF02 — Segurança e Privacidade:** Rotas administrativas protegidas por JWT. Consulta pública condicionada ao `booking_code` único, impedindo busca aberta por telefone.
- **RNF03 — Integridade Transacional:** Validação atômica no PostgreSQL durante a gravação do agendamento para evitar condições de corrida (*race conditions*).

---

## 5. Modelagem de Dados

| Tabela | Colunas Principais | Observação |
| :--- | :--- | :--- |
| **barbers** | `id`, `name`, `email`, `password_hash`, `is_active` | Profissionais com acesso ao painel. |
| **services** | `id`, `name`, `description`, `price`, `duration_minutes`, `is_active` | Catálogo de serviços. |
| **appointments** | `id`, `booking_code`, `client_name`, `client_phone`, `barber_id`, `start_time`, `end_time`, `total_price`, `status`, `notes` | `status`: `CONFIRMED`, `COMPLETED`, `CANCELED`, `BLOCKED`. |
| **appointment_services** | `appointment_id`, `service_id`, `price_snapshot`, `duration_snapshot` | Relacionamento N:N mantendo histórico de preço e duração da época do atendimento. |