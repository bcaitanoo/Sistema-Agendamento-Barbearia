# Sistema de Agendamento para Barbearia

Aplicação web full-stack desenvolvida para modernizar o agendamento de horários em barbearias. Permite que clientes escolham serviços, profissionais e horários disponíveis em tempo real, enquanto oferece aos barbeiros um painel administrativo para gestão de agenda e serviços.

> Para detalhes completos de requisitos e regras de negócio, consulte a [Especificação de Requisitos (SRS)](./docs/REQUISITOS.md).

---

## Sumário
- [Objetivo](#objetivo)
- [Tecnologias](#tecnologias)
- [Funcionalidades](#funcionalidades)
- [Regras de Negócio](#regras-de-negócio)
- [Licença](#licença)

---

## Objetivo
Eliminar conflitos de horários e agendamentos manuais desorganizados, entregando uma experiência rápida para o cliente (sem burocracia de cadastro) e controle operacional completo para a barbearia.

---

## Tecnologias
- **Front-end:** React e CSS
- **Back-end:** Python e FastAPI
- **Banco de Dados:** PostgreSQL

---

## Funcionalidades

### Área do Cliente
- **Catálogo de Serviços:** Visualização de serviços, valores e tempo de duração.
- **Seleção Múltipla:** Escolha de um ou mais serviços no mesmo agendamento com soma automática de valor e tempo.
- **Cálculo de Disponibilidade:** Exibição dinâmica apenas de horários livres conforme a duração total e profissional escolhido (incluindo opção *"Sem preferência"*).
- **Agendamento sem Login:** Reserva rápida informando apenas Nome e WhatsApp.
- **Consulta e Cancelamento:** Acompanhamento e cancelamento via código único gerado na reserva, com redirecionamento para WhatsApp caso esteja fora do prazo.

### Painel do Barbeiro
- **Autenticação:** Login seguro para acesso ao painel administrativo.
- **Gestão de Agenda:** Visualização da agenda individual ou geral da barbearia por data, com atualização de status (`Concluído` / `Cancelado`).
- **Encaixe Manual e Bloqueios:** Lançamento rápido de clientes de balcão e bloqueio de horários (almoço/folgas).
- **Gestão de Serviços:** Cadastro, edição e ativação/desativação de serviços, preços e durações.
- **Resumo Operacional:** Indicadores simples de atendimentos realizados e faturamento estimado do período.

---

## Regras de Negócio
- **Prevenção de Concorrência:** Validação no banco de dados impedindo que dois agendamentos ocupem o mesmo intervalo para o mesmo barbeiro.
- **Privacidade por Código Único:** Cada reserva gera um código exclusivo para consulta e cancelamento, evitando exposição de dados pessoais apenas pelo número de telefone.
- **Balanceamento Automático:** Na opção *"Sem preferência"*, o sistema aloca o cliente ao barbeiro com menor carga horária agendada no dia.

---

## Licença
Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](./LICENSE) para mais detalhes.
