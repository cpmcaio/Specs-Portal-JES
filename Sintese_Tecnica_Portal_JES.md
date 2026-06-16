# Síntese Técnica do Sistema
## Portal JES – Jogos do Ensino Superior
### Sistema de Gerenciamento Desportivo Universitário (SGDU)
**Instituto Federal da Paraíba – IFPB | Versão 1.0 | Junho de 2026**

---

## 1. Descrição do Sistema

O **Portal JES** (Jogos do Ensino Superior) é uma plataforma web desenvolvida para centralizar e automatizar a gestão integral das competições esportivas universitárias do IFPB. O sistema visa substituir fluxos manuais e descentralizados — como planilhas e formulários físicos — proporcionando uma redução estimada de **90% no tempo operacional** de inscrições e homologação de atletas.

Formalmente denominado **SGDU (Sistema de Gerenciamento Desportivo Universitário)**, o produto foi concebido como projeto de extensão institucional sem fins lucrativos, com potencial de licenciamento para outras Instituições de Ensino Superior (IES). Um pilar central do sistema é a conformidade com a **LGPD**, utilizando a metodologia **Privacy by Design (PbD)** para mitigar riscos de exposição de dados sensíveis dos estudantes — problema que afeta cerca de **79,4% das instituições de ensino brasileiras**.

### 1.1 Problemas Resolvidos

| Problema | Impacto | Afetados |
|---|---|---|
| Ineficiência operacional com processos manuais | Erros críticos e retrabalho | Organizadores e Capitães |
| Falta de transparência e histórico centralizado | Ausência de dados atualizados | Torcida, participantes e gestores |
| Vulnerabilidade de privacidade (LGPD) | Exposição de dados pessoais de discentes | Todos os usuários e o IFPB |
| Fraudes em inscrições | Dificuldade de interoperabilidade com bancos acadêmicos | Gestão do IFPB e TI |
| Atrasos em certificações | Erros no lançamento manual de frequências | Atletas e equipes |

### 1.2 Perfis de Usuário

| Ator | Nível | Responsabilidades |
|---|---|---|
| **Administrador** | N3 | Cadastro de moderadores, backup, manutenção geral do sistema |
| **Moderador** | N2 | Gerenciar atléticas, partidas, modalidades e atribuir cargos de Capitão |
| **Capitão** | N1 | Cadastrar atletas e criar equipes para modalidades da sua atlética |
| **Atleta / Visitante** | — | Consultar horários, resultados e tabelas na homepage pública |

---

## 2. Arquitetura do Sistema

O SGDU é construído sobre o framework **Django (Python)**, seguindo o padrão arquitetural **MVT (Model–View–Template)**. A arquitetura é modular, com separação clara de responsabilidades entre camadas, permitindo escalabilidade horizontal e facilidade de manutenção.

### 2.1 Padrão Arquitetural – MVT

- **Model** — camada de dados responsável pelas entidades (Atleta, Equipe, Atlética, Partida, Modalidade) e comunicação com o banco relacional via Django ORM.
- **View** — camada de negócio que processa requisições HTTP, aplica as regras de acesso (RBAC) e retorna respostas ao cliente.
- **Template** — camada de apresentação com renderização server-side, Tailwind CSS para estilização responsiva e auto-escaping nativo para proteção contra XSS.

### 2.2 Ambientes Operacionais

- **Ambiente do Usuário (Capitães/Atletas)** — acesso via navegadores modernos (Chrome, Firefox, Edge) em desktops e dispositivos móveis, com autenticação obrigatória por matrícula e senha.
- **Ambiente Administrativo (Moderadores/Admin)** — dashboard protegido com controle hierárquico de acesso, auditoria de dados sensíveis e estatísticas globais.
- **Ambiente de Desenvolvimento/Teste** — acesso restrito à equipe técnica e ao Product Owner, isolado do ambiente de produção.

### 2.3 Componentes e Integrações

- **Autenticação** — matrícula como identificador único (username), substituindo e-mail convencional.
- **Controle de Acesso** — RBAC com 3 níveis hierárquicos implementado nativamente no Django.
- **Banco de Dados** — banco relacional gerenciado exclusivamente via Django ORM, sem raw queries.
- **Integração Institucional** — interoperabilidade segura com o ecossistema **SUAP** para validação de vínculo acadêmico e elegibilidade dos atletas.
- **Caching e Desempenho** — previsão de uso de **Redis** e otimização de queries ORM para suportar picos de tráfego durante inscrições.
- **Homepage Pública** — interface sem autenticação exibindo estatísticas com atualização máxima de 1 dia, filtros e busca por partidas.

---

## 3. Requisitos Funcionais e Não Funcionais

### 3.1 Requisitos Funcionais

| ID | Descrição | Prioridade | Complexidade |
|---|---|---|---|
| **RF001** | Manter Atletas: cadastro pelo Capitão (nome, matrícula, foto, sexo) | Essencial | Média |
| **RF002** | Manter Capitão: Moderador promove atleta ao cargo de Capitão | Importante | Baixa |
| **RF003** | Manter Moderador: Admin cadastra Moderadores com permissão nível 2 | Essencial | Baixa |
| **RF004** | Manter Atléticas: Moderador cadastra atléticas (nome, cursos, capitão, instituto, campus) | Essencial | Média |
| **RF005** | Manter Modalidades: cadastro com limites mínimo e máximo de atletas por equipe | Essencial | Média |
| **RF006** | Manter Equipes: Capitão cria equipes associando atletas a modalidades da sua atlética | Essencial | Alta |
| **RF007** | Manter Partidas: Moderador gerencia partidas (equipes, data, horário, local, fase, status) | Essencial | Alta |
| **RF008** | Carregamento por Planilha: envio de planilhas para população em lote das partidas com validação automática | Importante | Alta |
| **RF009** | Otimização de Processos: integração com ecossistema institucional para automatizar inscrições manuais | Desejável | Alta |
| **RF010** | Primeiro Acesso: obrigatoriedade de troca de senha padrão no primeiro login | Essencial | Média |
| **RF011** | Homepage pública com estatísticas, filtros e busca para visitantes logados e anônimos | Importante | Média |

### 3.2 Regras de Negócio

- **RN001 – Matrícula** — apenas números, de 6 a 10 dígitos, única no sistema.
- **RN002 – Data de Nascimento** — não pode ser futura; idade estritamente entre 14 e 100 anos.
- **RN003 – Nome da Atlética** — único no sistema, entre 3 e 255 caracteres.
- **RN004 – Equipe** — nome único por atlética/modalidade; respeito ao limite de atletas; todos da mesma atlética; sem duplicatas na mesma modalidade.
- **RN005 – Súmula Automática** — gerada automaticamente ao criar uma partida, personalizada por modalidade.
- **RN006 – Validação de Planilha** — checagem de padrões de registros, tipos de esportes e algoritmo de abstração de dados para barrar erros de preenchimento.

### 3.3 Requisitos Não Funcionais

#### Segurança e Privacidade

| ID | Descrição |
|---|---|
| **RNF001** | Autenticação por matrícula como credencial exclusiva de identificação |
| **RNF002** | RBAC com 3 níveis hierárquicos: Capitão (N1), Moderador (N2), Admin (N3) |
| **RNF003** | Verificação de sessão em todas as views; mensagens de erro genéricas para não vazar dados |
| **RNF004** | Proteção anti-força-bruta com mensagem genérica "Credenciais Inválidas" sem revelar existência da matrícula |
| **RNF005** | Hashing de senhas com PBKDF2 + salt automático via `make_password` / `check_password` |
| **RNF006** | Senha forte: mín. 8 chars, ao menos 1 maiúscula, 1 minúscula, 1 número, sem senhas comuns |
| **RNF007** | Proteção CSRF com token em todos os formulários Django (`{% csrf_token %}`) |
| **RNF008** | Proteção SQL Injection via Django ORM; raw queries proibidas sem sanitização explícita |
| **RNF009** | Proteção XSS via auto-escaping nativo dos templates Django |
| **RNF010** | Conformidade total com LGPD no armazenamento e tratamento de dados de atletas |

#### Confiabilidade e Disponibilidade

| ID | Descrição |
|---|---|
| **RNF011** | Alta disponibilidade 24/7 durante todo o período das competições do JES |

#### Desempenho

| ID | Descrição |
|---|---|
| **RNF012** | Dados públicos na homepage com atraso máximo de 1 dia em relação aos eventos reais |

#### Interoperabilidade

| ID | Descrição |
|---|---|
| **RNF013** | Comunicação blindada com bancos de dados acadêmicos institucionais via protocolos seguros |
| **RNF014** | Exportação de relatórios em `.xlsx` ou `.csv` compatíveis com algoritmo estatístico adotado |

---

## 4. Tecnologias Utilizadas na Implementação

| Camada | Tecnologia / Ferramenta | Justificativa |
|---|---|---|
| **Backend** | Python + Django (MVT) | Framework maduro com ORM, CSRF, autenticação e admin nativos |
| **Frontend** | Tailwind CSS + Templates Django | Interface responsiva com auto-escaping nativo para proteção XSS |
| **Banco de Dados** | Banco relacional via Django ORM | Abstração completa de SQL; proteção nativa contra injeção |
| **Segurança de Senhas** | PBKDF2 + salt (`make_password` / `check_password`) | Padrão Django para hashing seguro com salt automático |
| **Caching / Performance** | Redis *(previsto)* | Cache de consultas frequentes para reduzir latência em picos de tráfego |
| **Integração Institucional** | SUAP (ecossistema IFPB) | Validação de matrícula e vínculo acadêmico em tempo real |
| **Exportação de Dados** | `.xlsx` / `.csv` | Interoperabilidade com algoritmos estatísticos externos |
| **Prototipação** | draw.io | Modelagem da arquitetura ambiental e fluxos do sistema |
| **Hospedagem (IES externas)** | Vercel, AWS ou DigitalOcean | Suporte a ambiente Python/Django; planos educacionais de baixo custo |

---

## 5. Vulnerabilidades OWASP Resolvidas

O sistema adota **Defense in Depth** (Defesa em Profundidade), combinando múltiplas camadas de proteção para endereçar as principais categorias do OWASP Top 10.

| Vulnerabilidade OWASP | Mitigação no Sistema | Requisito Vinculado |
|---|---|---|
| **A01 – Broken Access Control** | RBAC hierárquico (N1/N2/N3); verificação de sessão em todas as views; Capitão restrito à sua atlética | RNF002, RNF003 |
| **A02 – Cryptographic Failures** | Hashing PBKDF2 com salt automático para senhas; criptografia de dados sensíveis em repouso prevista no roadmap | RNF005 |
| **A03 – Injection (SQL)** | Uso exclusivo do Django ORM; raw queries proibidas sem sanitização explícita | RNF008 |
| **A04 – Insecure Design** | Privacy by Design (PbD) como metodologia central; RBAC nativo; integração segura com SUAP | RNF002, RNF010, RNF013 |
| **A05 – Security Misconfiguration** | Variáveis de ambiente para dados sensíveis; guia de instalação focado em segurança | RNF005, RNF013 |
| **A06 – Vulnerable Components** | Stack open-source auditável (Python/Django); arquitetura MVT modular e documentada | — |
| **A07 – Identification & Auth Failures** | Autenticação por matrícula + senha forte + troca obrigatória no primeiro acesso + proteção anti-brute-force | RNF001, RNF004, RNF006, RF010 |
| **A08 – Software & Data Integrity Failures** | Validação de planilhas com algoritmo de abstração de dados (RN006); súmulas geradas automaticamente (RN005) | RF008, RN005, RN006 |
| **A09 – Logging & Monitoring** | Auditoria de dados sensíveis para LGPD; fuso horário de Brasília para logs; histórico de atletas imutável | RNF010, Restrição 6 |
| **A10 – SSRF / Forgery** | Tokens CSRF em todos os formulários Django; comunicação blindada com sistemas institucionais | RNF007, RNF013 |

---

## 6. Questões Técnicas Importantes

### 6.1 Conformidade com LGPD (Privacy by Design)

A conformidade com a LGPD é tratada como **restrição crítica**, não como funcionalidade adicional. Os princípios aplicados são:

- **Minimização de Dados** — apenas os dados estritamente necessários para fins desportivos são coletados (nome, matrícula, sexo, foto).
- **Controle de Acesso Granular** — Capitães acessam apenas dados de sua própria atlética, sem visibilidade de outras entidades.
- **Mensagens Genéricas** — erros de login nunca revelam se a matrícula existe no banco, prevenindo enumeração de usuários.
- **Imutabilidade de Registros** — atletas que participaram de competições homologadas não podem ser excluídos, garantindo auditabilidade histórica.
- **Auditoria e Logs** — todos os registros utilizam o fuso horário oficial de Brasília para rastreabilidade completa.

### 6.2 Modelo de Controle de Acesso (RBAC)

O sistema implementa três níveis hierárquicos de permissão:

- **Nível 1 – Capitão** — acesso restrito à sua atlética. Pode cadastrar atletas e criar equipes. Sem visibilidade de outras atléticas ou configurações globais.
- **Nível 2 – Moderador** — acesso global a todas as atléticas. Gerencia modalidades, partidas, equipes e atribui cargos de Capitão. Sem acesso às funções administrativas do sistema.
- **Nível 3 – Administrador** — acesso total. Cadastra Moderadores, realiza backup, gerencia configurações globais e visualiza estatísticas estratégicas.

> A dashboard só é acessível por usuários credenciados. A homepage pública (estatísticas) é a única interface disponível sem autenticação.

### 6.3 Primeiro Acesso e Gestão de Senhas

O fluxo de primeiro acesso é crítico para a segurança operacional do sistema:

- **Senha Padrão** — Moderadores são cadastrados pelo Admin com uma senha padrão definida na criação.
- **Troca Obrigatória** — no primeiro login, o sistema bloqueia o acesso até que o usuário defina uma nova senha (RF010).
- **Política de Senha Forte** — mínimo 8 caracteres, ao menos 1 maiúscula, 1 minúscula, 1 número, sem constar em blacklists de senhas comuns (RNF006).
- **Proteção Anti-Brute Force** — mensagens genéricas que jamais revelam se a matrícula existe no sistema (RNF004).

### 6.4 Integração e Validação de Planilhas

O carregamento de dados por planilha (RF008) é uma funcionalidade de alto risco operacional. O sistema implementa uma camada de validação robusta (RN006) que:

- Verifica padrões de registros e formatos esperados por modalidade.
- Aplica um algoritmo de abstração de dados para detectar e barrar erros manuais de árbitros.
- Valida os tipos de esportes referenciados na planilha contra os cadastros existentes no sistema.
- Restringe o upload exclusivamente a Administradores e Moderadores.

### 6.5 Geração Automática de Súmulas (RN005)

Ao registrar uma nova partida (RF007), o sistema gera automaticamente uma **súmula personalizada** para aquela modalidade específica. Esse documento oficial registra tudo que ocorre antes, durante e após a partida, garantindo rastreabilidade e eliminando o risco de omissões no preenchimento manual pós-jogo.

### 6.6 Escalabilidade e Disponibilidade

- **Alta Disponibilidade** — operação 24/7 durante o período das competições (RNF011).
- **Escalabilidade Horizontal** — arquitetura modular Django permite balanceamento de carga e deploy em cloud (AWS, DigitalOcean) para picos de inscrições.
- **Otimização de Consultas** — uso consciente do ORM com previsão de caching Redis para consultas frequentes.
- **Portabilidade** — instalação autônoma pelo setor de TI de qualquer IES que disponha de ambiente Python/Django e banco de dados relacional.

### 6.7 Potencial Comercial (SaaS)

Embora desenvolvido como projeto de extensão sem fins lucrativos para o IFPB, o SGDU possui estrutura para expansão comercial:

- **Licenciamento SaaS para IES** — modelo de assinatura para universidades e Institutos Federais com processos ainda manuais.
- **Módulo de Analytics** — painel de inteligência de dados para reitorias e coordenadorias esportivas.
- **Módulos de Integração via API** — conectores licenciáveis para integração com sistemas acadêmicos além do SUAP.
- **Consultoria LGPD / Privacy by Design** — serviço técnico para adaptação de sistemas legados de outras IES.

---

*Documento de Síntese Técnica – Portal JES / SGDU*  
*Instituto Federal da Paraíba – IFPB | Junho 2026*
