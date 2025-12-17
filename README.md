# **EducaOnline – Plataforma Modular de Gestão de Cursos e Alunos**

## **1. Visão Geral**

O **EducaOnline** é uma plataforma corporativa distribuída, construída com **arquitetura de microsserviços**, para **gestão completa de cursos, alunos, matrículas, pagamentos e certificados**, integrando múltiplos domínios via **mensageria RabbitMQ** e um **BFF (Backend for Frontend)** em .NET e um **frontend em Angular**.

O projeto foi desenvolvido como parte do MBA **DevXpert Full Stack .NET**, no módulo **Construção de Aplicações Corporativas**, aplicando **DDD (Domain-Driven Design)**, **CQRS**, **Event-Driven Architecture** e **Clean Architecture**.

---

## **2. Autor**

- **Victor Lino**

---

## **3. Arquitetura do Projeto**

A solução é organizada em **camadas independentes**:

```
EducaOnline/
│
├── backend/
│   └── src/
│       ├── ApiGateways/
│       │   └── EducaOnline.Bff/                  → BFF central que integra os domínios e o frontend
│       ├── BuildingBlocks/
│       │   ├── EducaOnline.Core/                 → Domínios compartilhados, validações, eventos
│       │   ├── EducaOnline.MessageBus/           → Implementação do barramento RabbitMQ
│       │   └── EducaOnline.WebAPI.Core/          → Middlewares, JWT e extensões de API
│       ├── Services/
│       │   ├── Aluno/
│       │   │   └── EducaOnline.Aluno.API/        → Contexto de alunos, matrículas e certificados
│       │   ├── Conteudo/
│       │   │   └── EducaOnline.Conteudo.API/     → Contexto de cursos e aulas
│       │   ├── Financeiro/
│       │   │   ├── EducaOnline.Financeiro.API/   → Contexto de faturamento e pagamentos
│       │   │   └── EducaOnline.Financeiro.Pagamentos/
│       │   ├── Identidade/
│       │   │   └── EducaOnline.Identidade.API/   → Autenticação e autorização (JWT)
│       │   └── Pedidos/
│       │       └── EducaOnline.Pedidos.API/      → Contexto de pedidos e integração financeira
│
└── frontend/
│    ├── apps/                                     → Aplicações Angular (Portal do Aluno, Admin, etc.)
│    ├── libs/                                     → Módulos e componentes compartilhados
│    ├── package.json                              → Configuração de dependências
└── README.md                                 → Documentação específica do frontend
```
---

## **4. Comunicação entre Domínios**

O **Message Bus** (RabbitMQ) é utilizado para a troca de eventos entre os contextos:

- **Identidade** → publica criação de usuários (Aluno/Admin)
- **Aluno.API** → publica matrícula criada / concluída
- **Financeiro.API** → publica eventos de pagamento
- **Pedidos.API** → publica confirmação de pedido
- **Conteudo.API** → Criar cursos, aulas e conteúdo programático

**Configuração de conexão RabbitMQ (appsettings.json):**
```json
"MessageQueueConnection": {
  "MessageBus": "host=localhost:5672;publisherConfirms=true;timeout=10"
}
```
---

## **5. Autenticação e Autorização**

- Implementada via **ASP.NET Core Identity** + **JWT Bearer Tokens**
- **Identidade.API** é o ponto central de autenticação e emissão de tokens
- Cada microserviço valida JWT via `EducaOnline.WebAPI.Core.Identidade.JwtConfiguration`
- Tokens são obrigatórios para todas as rotas `[Authorize]`

---

## **6. Domínios e Funcionalidades**

| Domínio | Responsabilidades Principais |
|----------|-------------------------------|
| **Identidade.API** | Registro e autenticação de usuários, seed automático de Admin e Aluno |
| **Aluno.API** | Gerenciamento de alunos, matrículas, progresso e certificados |
| **Conteudo.API** | Criação e manutenção de cursos, aulas e conteúdo programático |
| **Financeiro.API** | Processamento de pagamentos, controle financeiro |
| **Pedidos.API** | Orquestração de pedidos e integração com pagamento |
| **BFF (EducaOnline.Bff)** | Intermediação entre frontend e APIs — matrícula e checkout |
| **Frontend Angular** | Interface web do aluno e do administrador |
---

## **7. Seeds Automáticos**

Cada domínio realiza **seed automático em ambiente Development**:

- **Identidade:**  
  Cria usuários `admin@educaonline.com.br` e `aluno@educaonline.com.br`  
  → `Aluno` possui o `Guid` fixo `40640fec-5daf-4956-b1c0-2fde87717b66`

- **Conteúdo:**  
  Cria 3 cursos (IA, Angular, .NET)

- **Aluno:**  
  Cria o aluno com mesmo ID do Identity e matrícula automática nos 3 cursos

---

## **8. Executando o Projeto**

### **Pré-requisitos**
- .NET SDK 9.0 ou superior  
- Node.js 20+
- Docker Desktop  
- Visual Studio 2022 ou VS Code  
- SQLite  

---

### **1️ Iniciar o RabbitMQ via Docker**
```bash
docker run -d --hostname educa-rabbit --name educa-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management
```
Acesse o painel RabbitMQ em:  
http://localhost:15672  
Usuário/Senha padrão: `guest / guest`

---

### **2️ Executar a Solução**

#### A partir do Visual Studio:
- Defina o projeto **EducaOnline.Bff** como Startup Project.
- Certifique-se de que as APIs `Identidade`, `Aluno`, `Conteudo`, `Pedidos` e `Financeiro` também estão com **Multiple Startup Projects**.

#### A partir do terminal:
```bash
cd src/Services/EducaOnline.Identidade.API
dotnet run

cd src/Services/EducaOnline.Aluno.API
dotnet run

cd src/Services/EducaOnline.Conteudo.API
dotnet run

cd src/Services/EducaOnline.Pedidos.API
dotnet run

cd src/ApiGateways/EducaOnline.Bff
dotnet run
```
### **3 Executar o Frontend Angular**
```bash
cd frontend
npm install
nx s educa-online
```
Acesse em: http://localhost:4200
---

## **9. Autenticação via Swagger**

1. Acesse o Swagger da API de **Identidade**
   - URL: `https://localhost:7001/swagger`
2. Realize login do aluno com:
   - **Usuário:** aluno@educaonline.com.br  
   - **Senha:** Teste@123  
3. Realize login do admin com:
   - **Usuário:** admin@educaonline.com.br  
   - **Senha:** Teste@123  
4. Copie o JWT retornado e insira no botão **“Authorize”** dos outros serviços.

---

## **10. Documentação Técnica**

- **Padrões Utilizados:**
  - DDD (Domain Driven Design)
  - CQRS e MediatR
  - Repository + Unit of Work
  - Value Objects e Entidades Ricas
  - AutoMapper e FluentValidation

- **Camadas:**
  - **Application** → Handlers e Commands
  - **Domain** → Entidades e Eventos
  - **Infra** → Contexto EF e Repositórios
  - **API** → Exposição de Endpoints RESTful

---

## **11. Serviçoes e Portas**

| Projeto                        | Porta | Descrição                |
| ------------------------------ | ----- | ------------------------ |
| **Frontend Angular**           | 4200  | Interface web            |
| **EducaOnline.Bff**            | 7000  | Gateway de integração    |
| **EducaOnline.Identidade.API** | 7001  | Autenticação JWT         |
| **EducaOnline.Conteudo.API**   | 7002  | Cursos e aulas           |
| **EducaOnline.Aluno.API**      | 7003  | Alunos e matrículas      |
| **EducaOnline.Pedidos.API**    | 7004  | Pedidos e checkout       |
| **EducaOnline.Financeiro.API** | 7005  | Processamento financeiro |

## **12. Licença e Contribuição**

Este projeto é parte de um curso acadêmico e **não aceita contribuições externas**.

Para dúvidas ou sugestões, utilize a aba **Issues** no GitHub.
---
