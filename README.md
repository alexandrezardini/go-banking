# GO Banking

**Disclaimer:** GO Banking é um projeto experimental para fins de estudo e teste. O nome é utilizado apenas neste contexto de aprendizado.

## Visão Geral

GO Banking é um serviço backend que simula uma plataforma financeira, permitindo que usuários realizem transferências entre contas de qualquer banco no Brasil, utilizando Open Finance e Pix. O serviço fornece uma visão consolidada dos saldos dos usuários em diferentes bancos e possibilita transferências entre contas.

## Objetivos

O principal objetivo deste projeto é explorar e aprender sobre desenvolvimento backend, design de sistemas e tecnologias modernas como microserviços, arquitetura orientada a eventos, Kubernetes, Docker e serviços da AWS.

## Arquitetura Detalhada

A arquitetura é baseada em microserviços, hospedada na AWS, containerizada com Docker e orquestrada com Kubernetes (AWS EKS). Cada componente é implementado como um serviço independente em um contêiner Docker, implantado em pods dentro do cluster Kubernetes.

### Componentes e Serviços AWS

#### AWS Elastic Kubernetes Service (EKS)

- **Descrição**: Serviço gerenciado de Kubernetes que facilita a execução de clusters Kubernetes na AWS.
- **Uso no Projeto**: Hospeda o cluster Kubernetes onde os contêineres Docker dos microserviços são implantados.

#### Docker Containers

- Cada microserviço é empacotado em um contêiner Docker para garantir consistência e portabilidade.
- **Contêineres**:
  - **API Gateway Container**: Gerencia as solicitações dos clientes e roteia para os serviços apropriados.
  - **Auth Service Container**: Responsável pela autenticação e autorização dos usuários.
  - **Account Service Container**: Gerencia informações das contas e saldos dos usuários.
  - **Transaction Service Container**: Lida com a criação e processamento das transações.
  - **Event Consumer Containers**: Consumidores que processam eventos do Kafka.

#### AWS Managed Streaming for Apache Kafka (Amazon MSK)

- **Descrição**: Serviço gerenciado para executar clusters Apache Kafka na AWS.
- **Uso no Projeto**: Serve como o barramento de eventos para comunicação orientada a eventos entre os serviços.

#### Amazon DynamoDB

- **Descrição**: Banco de dados NoSQL rápido e flexível.
- **Uso no Projeto**: Armazena dados de contas, transações e eventos.

#### AWS CloudWatch

- **Descrição**: Serviço de monitoramento para recursos AWS e aplicações.
- **Uso no Projeto**: Monitoramento e logging dos recursos AWS e aplicações em execução.

#### AWS IAM

- **Descrição**: Gerenciamento de acesso seguro aos serviços e recursos da AWS.
- **Uso no Projeto**: Controle de acesso e políticas de segurança para recursos AWS.

#### AWS Elastic Load Balancing (ELB)

- **Descrição**: Distribui automaticamente o tráfego de entrada entre múltiplas instâncias.
- **Uso no Projeto**: Distribui o tráfego para os nós do EKS e balanceia a carga entre os pods.

#### Amazon Route 53

- **Descrição**: Serviço de DNS escalável e altamente disponível.
- **Uso no Projeto**: Gerencia o roteamento de tráfego para o ELB e serviços.

#### AWS Key Management Service (KMS)

- **Descrição**: Serviço gerenciado para criação e controle de chaves de criptografia.
- **Uso no Projeto**: Gerencia as chaves de criptografia para dados em repouso.

### Arquitetura Lógica

A seguir, detalhamos como os componentes interagem dentro da arquitetura do sistema.

#### Fluxo de Dados

1. **Cliente (Usuário)**: Inicia solicitações através de aplicativos cliente (web/mobile).

2. **Amazon Route 53**: Resolve o domínio para o ELB.

3. **Elastic Load Balancer (ELB)**: Distribui o tráfego para os nós do cluster EKS.

4. **AWS EKS Cluster**:

   - **API Gateway**:
     - Recebe as solicitações dos usuários.
     - Roteia as solicitações para os microserviços apropriados.
   - **Auth Service**:
     - Autentica e autoriza usuários.
   - **Account Service**:
     - Gerencia informações de contas e saldos.
   - **Transaction Service**:
     - Processa transações e interage com o Kafka para publicar eventos.

5. **Amazon MSK (Kafka)**:

   - **Produtores**: Microserviços que publicam eventos (e.g., Transaction Service).
   - **Consumidores**: Microserviços que consomem eventos (e.g., Account Service para atualizar saldos).

6. **Amazon DynamoDB**:

   - Armazena dados persistentes como contas, transações e eventos.

7. **AWS CloudWatch**:

   - Coleta logs e métricas dos serviços.
   - Configurado para alertas e dashboards.

### Estratégia de Sharding no Kubernetes

- **Namespaces**:

  - Separação lógica dos ambientes (e.g., dev, staging, prod).
  - Permite gerenciamento isolado de recursos e políticas.

- **Node Groups**:

  - Grupos de nós com configurações específicas (e.g., tamanho de instância, capacidade).
  - Podem ser usados para separar cargas de trabalho (e.g., serviços críticos vs. serviços auxiliares).

- **Horizontal Pod Autoscaling (HPA)**:

  - Ajusta automaticamente o número de pods com base em métricas (e.g., uso de CPU, custom metrics).
  - Garante que o serviço possa escalar horizontalmente conforme a demanda.

### Diagrama Detalhado da Arquitetura
![design](go-banking-design.png)