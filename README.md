# Taskboard

Sistema de gerenciamento de tarefas no estilo Kanban com persistência relacional. Funciona via linha de comando (CLI) e organiza tarefas em colunas dentro de boards configuráveis. Projeto pessoal desenvolvido durante minha transição de Técnico de TI para desenvolvedor backend Java.

## Sobre o projeto

Cada board representa um quadro Kanban com três colunas padrão:

- **Inicial** — onde a tarefa nasce
- **Final** — onde a tarefa termina seu fluxo
- **Cancelada** — descarte explícito

As tarefas trafegam entre colunas, podem ser bloqueadas com motivo registrado e desbloqueadas com motivo registrado (auditoria simples). Toda a estrutura é persistida em MySQL com integridade referencial garantida por foreign keys e índices em campos de busca.

## Stack

| Camada | Tecnologia |
|---|---|
| Linguagem | Java 24 |
| Framework | Spring Boot 3.5.5 |
| Persistência | Spring Data JPA + Hibernate |
| Banco | MySQL 8 |
| Connection pool | HikariCP |
| Build | Maven |

## Arquitetura

```
src/main/java/com/rian/taskboard/taskboard
├── model           → Board, BoardColumn, ColumnType (enum), Task
├── repository      → BoardRepository, BoardColumnRepository, TaskRepository
├── service         → BoardService (regras de domínio, @Transactional)
├── TaskboardApp    → loop CLI
└── TaskboardApplication → bootstrap Spring Boot

src/main/resources
├── application.properties
└── db/migration/V1__create_board_tables.sql
```

### Modelo relacional

```
┌─────────┐ 1     N ┌──────────────┐ 1     N ┌──────┐
│  board  │─────────│ board_column │─────────│ task │
└─────────┘         └──────────────┘         └──────┘
```

- `board` → contém colunas
- `board_column` → tipo `INITIAL`, `FINAL` ou `CANCEL`, ordenada por `column_order`
- `task` → vinculada a uma coluna, com campos de bloqueio (`blocked`, `block_reason`, `unblock_reason`)
- Índices em `(board_id, column_order)` e em `column_id` para acelerar listagens

## Funcionalidades

- Criar board com colunas padrão
- Listar e selecionar boards existentes
- Criar tarefas em colunas
- Listar tarefas por coluna
- Mover tarefas entre colunas
- Bloquear tarefa com motivo
- Desbloquear tarefa com motivo
- Cancelar tarefas
- Persistência completa em MySQL

## Como rodar localmente

### Pré-requisitos

- Java JDK 24+
- MySQL 8 rodando na porta `3307` (ou ajustar `application.properties`)
- Maven (ou usar o `mvnw` incluso)

### Passo a passo

1. **Clone o repositório**
   ```bash
   git clone https://github.com/riansilva-dev/taskboard.git
   cd taskboard
   ```

2. **Configure o banco**

   Crie o database `taskdb` no seu MySQL e ajuste credenciais em `src/main/resources/application.properties` se necessário. As tabelas são criadas automaticamente na primeira execução (`ddl-auto=update`), e o script SQL de referência está em `db/migration/V1__create_board_tables.sql`.

3. **Execute**
   ```bash
   ./mvnw spring-boot:run
   # ou no Windows
   mvnw.cmd spring-boot:run
   ```

A aplicação inicia em modo CLI e exibe o menu interativo no terminal.

## Decisões técnicas

- **CLI ao invés de REST** — projeto focado em modelar o domínio Kanban e o ciclo de vida de tarefas (criação, movimento, bloqueio, cancelamento). A escolha por CLI mantém o escopo no domínio sem distração com camadas HTTP.
- **`@Transactional` na service** — operações que envolvem múltiplas entidades (criar board com colunas) são atômicas.
- **`fetch = LAZY` em relacionamentos** — evita carregar grafos inteiros desnecessariamente.
- **Migration SQL versionada** — `V1__create_board_tables.sql` documenta o schema esperado para revisão e auditoria, mesmo que o Hibernate atualize automaticamente em desenvolvimento.

## Próximos passos

- [ ] Exposição via API REST com Spring Web
- [ ] Testes unitários da camada de serviço (JUnit + Mockito)
- [ ] Testes de integração com Testcontainers
- [ ] Flyway para gerenciar migrations em produção
- [ ] Autenticação básica para múltiplos usuários

## Contato

Construído por **Rian Silva** — em transição de Técnico de TI N4 para desenvolvedor backend Java.

📧 riansilvasantos04@gmail.com · 🔗 [LinkedIn](https://www.linkedin.com/in/riansilva-dev/) · 💻 [GitHub](https://github.com/riansilva-dev)
