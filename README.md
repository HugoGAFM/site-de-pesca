# Projeto Pescarias — Documentação Técnica

Este documento descreve o sistema, sua arquitetura, diagramas, padrões aplicados e evidências de boas práticas. Use os arquivos em `docs/` (PlantUML) para gerar diagramas visuais.

**Sumário**
- Descrição do sistema
- Arquitetura
- Diagrama de classes (PlantUML)
- Diagrama de banco de dados (PlantUML)
- Padrões de projeto aplicados
- Evidências de boas práticas (Clean Code, SOLID)
- Como rodar e diagnosticar
- Endpoints importantes
- Observações e próximos passos

---

## 1. Descrição do sistema

"Pescarias" é um e‑commerce didático para produtos de pesca, composto por:
- Backend: Spring Boot (Java) com autenticação JWT, JPA/Hibernate para persistência em SQLite.
- Frontend: páginas estáticas HTML + JS (bootstrap) que consomem as APIs REST do backend.

Funcionalidades principais:
- Cadastro e login de usuários (JWT emitido pelo servidor).
- Criação de pedidos (checkout) associados ao usuário autenticado.
- Listagem de pedidos do usuário.

Objetivo do projeto: ser um exercício prático de integração entre frontend estático e backend REST com autenticação, persistência e boas práticas de projeto em Java/Spring.

---

## 2. Arquitetura

Camadas principais:
- Apresentação: páginas estáticas em `frontend/` (login, register, checkout, perfil, etc.).
- API: controladores Spring REST localizados em `backend/.../controller/` (ex.: `AuthController`, `PedidoController`).
- Negócio: entidades e DTOs (`entities/`, `dto/`) e serviços/utilitários (ex.: `TokenConfig`).
- Persistência: Spring Data JPA (`repository/`) com SQLite (`jdbc:sqlite:banco.db`).
- Segurança: Spring Security com filtro JWT (`SecurityFilter`, `SecurityConfig`).

Comunicação: o frontend faz `fetch` para `http://localhost:8080` (endpoints `/auth/login`, `/auth/register`, `/pedidos`, ...).

---

## 3. Diagrama de classes (PlantUML)

Veja `docs/class-diagram.puml`. Ele contém as principais classes/entidades:
- `User` (entidade JPA, implementa UserDetails)
- `Pedido` (entidade que referencia `User`)
- DTOs: `PedidoRequest`, `LoginRequest`, `RegisterUserRequest`, `LoginResponse`
- `AuthController`, `PedidoController`, `SecurityFilter`, `TokenConfig`

(Use PlantUML para renderizar.)

---

## 4. Diagrama de banco de dados (PlantUML)

Veja `docs/db-diagram.puml`. Tabelas principais:
- `tb_user` (id, nome, sobrenome, username, senha, role, email)
- `tb_pedido` (id, data, produto, preco, user_id)

Relação: `tb_pedido.user_id` -> `tb_user.id` (ManyToOne)

---

## 5. Padrões de projeto aplicados

- DTO (Data Transfer Object): os controladores aceitam/retornam objetos simples (ex.: `PedidoRequest`) para separar contrato HTTP da entidade persistida.
- Repository pattern: uso de `JpaRepository` para acesso a dados (`UserRepository`, `PedidoRepository`).
- Dependency Injection: controladores e repositórios injetados via construtor (padrão Spring).
- Strategy (Security filter): a validação do token é encapsulada no `SecurityFilter` e `TokenConfig` administra geração/validação de tokens.

Observação: há espaço para aplicar Services (camada de serviço) para lógica de negócios, evitando que controllers façam muita lógica.

---

## 6. Evidências de boas práticas (Clean Code, SOLID)

- Single Responsibility (S do SOLID): as entidades modelam dados; controllers lidam principalmente com entrada/saída; `TokenConfig` cuida de tokens.
- Open/Closed: componentes como `TokenConfig` e `SecurityFilter` podem ser estendidos/alterados sem modificar contratos dos controllers.
- Liskov/Interface Segregation/Dependency Inversion: uso de interfaces Spring Data e abstrações do framework facilita substituição de implementações.
- Clean Code:
  - Nomes claros de classes e métodos (`PedidoController.criarPedido`, `AuthController.login`).
  - Pequenas funções JS nas páginas para inicializar menu e tratar token.
  - Logs informativos (logger.info nos controllers).

Melhorias recomendadas (para seguir ainda mais os princípios):
- Introduzir uma camada `Service` (ex.: `PedidoService`) para encapsular lógica de negócio e reduzir responsabilidades dos controllers.
- Não retornar entidades JPA diretamente em respostas públicas — use DTOs/Response objects (evita problemas com proxies e expõe apenas campos necessários).
- Externalizar segredos (JWT secret) para variáveis de ambiente/config segura.

---

## 7. Como rodar (resumo)

Pré‑requisitos:
- Java JDK (11+ recomendado, JDK 17 ou 23 no ambiente do desenvolvedor)
- Gradle (wrapper incluso, use `./gradlew` / `.
` no Windows `.
`)

Rodando o backend (Windows PowerShell):
```powershell
cd backend/site-de-pesca/site-de-pesca
.\gradlew bootRun
```

Frontend: as páginas são estáticas, abra `frontend/home.html` no navegador (ou sirva com um servidor estático se preferir).

Banco de dados: `banco.db` fica na pasta do backend (caminho relativo definido em `application.properties`).

---

## 8. Endpoints importantes

- `POST /auth/register` — corpo `RegisterUserRequest` → cria usuário
- `POST /auth/login` — corpo `LoginRequest` → retorna token JWT (no response `LoginResponse` com token)
- `POST /pedidos` — cria pedido (Authorization: Bearer <token>)
- `GET /pedidos` — lista pedidos do usuário autenticado (Authorization: Bearer <token>)

Tokens: armazenados pelo frontend em `localStorage` sob a chave `token`.

---

## 9. Diagnósticos comuns

- Se `GET /pedidos` retornar 500 por causa de Jackson/Hibernate proxy: solução aplicada — `@JsonIgnoreProperties({"hibernateLazyInitializer","handler"})` no `User` ou mapear para DTOs.
- Se o DB Browser mostrar vazio: verifique o `banco.db` correto (veja `application.properties`) e pare o backend antes de abrir, ou copie o arquivo para inspecionar.

---

## 10. Arquivos úteis no projeto

- Backend:
  - `src/main/java/.../config/` — `SecurityConfig`, `SecurityFilter`, `TokenConfig`
  - `src/main/java/.../controller/` — `AuthController`, `PedidoController`, `TestController`
  - `src/main/java/.../entities/` — `User`, `Pedido`, `Produto` (quando aplicável)
  - `src/main/resources/application.properties` — configura datasource, logging
- Frontend: `frontend/` com `home.html`, `login.html`, `register.html`, `checkout.html`, `perfil.html`, `menu.html`

---

