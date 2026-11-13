# Guia de Boas Práticas: Mitigação contra Força Bruta, Enumeração e Password Spraying

## 1. Introdução

Ataques de autenticação continuam sendo um dos vetores mais explorados por agentes maliciosos para obter acesso inicial à infraestrutura corporativa. Entre as abordagens mais comuns, destacam-se:

* **Enumeração de Serviços e Usuários:** Identificação de portas abertas, serviços expostos e nomes de contas válidas na organização.
* **Força Bruta (*Brute Force*):** Tentativas exaustivas de adivinhar combinações de usuário e senha por meio de listas ou dicionários (*wordlists*).
* **Pulverização de Senhas (*Password Spraying*):** Estratégia em que o invasor testa poucas senhas comuns (ex.: `Empresa@2026`) contra uma vasta lista de usuários diferentes, evitando deliberadamente os limites tradicionais de bloqueio de conta (*account lockout*).

Este documento detalha medidas técnicas, operacionais e arquiteturais recomendadas para proteger ambientes corporativos contra essas ameaças.

---

## 2. Estratégias de Proteção e Defesa

```
                                [ Vetores de Ataque ]
               (Enumeração)      (Password Spraying)       (Força Bruta)
                    │                     │                      │
                    ▼                     ▼                      ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                        Camadas de Defesa Corporativa                        │
│                                                                             │
│  [ Rede / Borda ]        Rate Limiting, WAF, Ocultação de Banners           │
│  [ Autenticação ]        MFA / FIDO2, Política de Passphrases Longas        │
│  [ Gestão de Contas ]    Ciclo de Vida com Expiração, Menor Privilégio      │
│  [ Monitoramento ]       SIEM, Bloqueio Adaptativo Baseado em Risco         │
└─────────────────────────────────────────────────────────────────────────────┘
                                          │
                                          ▼
                             [ Acesso Seguro Garantido ]
```

---

### Medida 1: Políticas Modernas de Senhas (Comprimento e Passphrases vs. Complexidade Arbitrária)

#### A Abordagem Recomendada
Historicamente, organizações exigiam senhas curtas (8 caracteres) com símbolos aleatórios, o que levava os usuários a adotarem padrões previsíveis (ex.: `Mudar123!`). O padrão moderno recomendado pelo **NIST (SP 800-63B)** prioriza o **comprimento** por meio de **Passphrases** (frases de acesso com 4 ou mais palavras não relacionadas, ex.: `café-nuvem-janela-azul-94`).

#### Como Auxilia na Proteção
* **Resistência Matemática à Força Bruta:** O tempo necessário para computar e testar combinações cresce exponencialmente com o número de caracteres. Enquanto uma senha de 8 caracteres alfanuméricos pode ser quebrada em minutos por ataques offline com GPUs modernas, uma frase de acesso com 16 a 20 caracteres torna a força bruta computacionalmente inviável.
* **Bloqueio de Senhas Conhecidas:** Integração do Active Directory / IdP com bases de credenciais vazadas (ex.: APIs do *Have I Been Pwned* ou proteção de credenciais do Microsoft Entra ID), impedindo que colaboradores definam senhas presentes em wordlists de ataque.

---

### Medida 2: Gestão do Ciclo de Vida de Credenciais e Prazos de Validade

#### A Abordagem Recomendada
Em vez de forçar trocas frequentes de senhas a cada 30 ou 60 dias (o que comprovadamente incentiva senhas fracas e previsíveis), a organização deve adotar **expiração orientada a contexto e ciclo de vida**:
1. **Contas Temporárias / Terceirizados:** Obrigatoriedade de data final de expiração pré-programada no momento da criação da conta.
2. **Tokens de Acesso e Chaves de API:** Rotação estrita e automatizada de credenciais de serviço e chaves de integração.
3. **Revogação Imediata no Desligamento:** Automação via RH/IDM para suspensão imediata de contas no encerramento de contratos.

#### Como Auxilia na Proteção
* **Eliminação de Contas Zumbis (*Dormant Accounts*):** Contas órfãs ou esquecidas de ex-funcionários e prestadores de serviços são alvos primários em campanhas de *Password Spraying*, pois muitas vezes não possuem donos monitorando tentativas de login.
* **Redução da Janela de Exposição:** Caso uma credencial seja comprometida sem o conhecimento da equipe de segurança, o prazo de validade finito da conta ou do token limita o tempo de permanência do invasor na rede.

---

### Medida 3: Autenticação Multifator Obrigatória (MFA / 2FA) e FIDO2

#### A Abordagem Recomendada
Tornar obrigatória a exigência de um segundo fator de autenticação em **todos** os pontos de acesso externos (VPN, Webmail, Portais SaaS, SSH e RDP). Priorizar fatores resistentes a phishing, como chaves de segurança de hardware (FIDO2/WebAuthn) ou aplicativos com confirmação de número (*number matching*), abandonando SMS e chamadas de voz.

#### Como Auxilia na Proteção
* **Neutralização Imediata de Credenciais Vazadas:** Mesmo que um ataque de força bruta ou spray descubra o par correto de usuário e senha, o acesso é barrado na barreira do segundo fator.
* **Alerta Prévio:** Notificações de MFA inesperadas funcionam como alerta imediato ao colaborador e ao SOC de que uma senha válida está sendo usada por terceiros não autorizados.

---

### Medida 4: Limitação de Taxa (*Rate Limiting*) e Bloqueio Inteligente/Adaptativo

#### A Abordagem Recomendada
Adoção de sistemas de bloqueio adaptativo (*Smart Lockout*) em substituição ao bloqueio rígido por conta:
* Em vez de travar a conta do usuário após 5 erros (o que causaria Negação de Serviço proposital pelo atacante), o sistema analisa a origem da requisição (endereço IP, reputação da rede, geolocalização e padrão de navegação).
* Imposição de atrasos progressivos (*exponential backoff* / tarpitting) entre tentativas incorretas.

#### Como Auxilia na Proteção
* **Defesa contra Password Spraying:** O atacante que espalha uma única senha por centenas de contas a partir do mesmo bloco de IPs é bloqueado pelo volume global de requisições, sem impedir os colaboradores legítimos de trabalhar.
* **Desaceleração de Ataques Automatizados:** Aumentar artificialmente o tempo de resposta do servidor a cada erro inviabiliza ferramentas automatizadas como Hydra ou Medusa, tornando o ataque inviável em termos de tempo.

---

### Medida 5: Hardening de Superfície de Ataque e Prevenção de Enumeração

#### A Abordagem Recomendada
* **Ocultação de Mensagens Informativas:** Mensagens de erro de login genéricas (ex.: `"Credenciais inválidas"` em vez de `"Usuário não encontrado"` ou `"Senha incorreta"`), impedindo que o atacante confirme quais e-mails ou nomes de usuário existem no sistema.
* **Ocultação de Versões e Banners:** Desativar assinaturas de servidores web, banners de FTP/SSH e interfaces de administração expostas diretamente para a internet pública sem VPN.
* **Isolamento de Protocolos Legados:** Desativação de protocolos inseguros de texto claro (FTP, Telnet, POP3/IMAP sem TLS, SMBv1) e restrição de acessos administrativos por IP ou salto controlado (*Bastion Host*).

#### Como Auxilia na Proteção
* **Cegueira para o Invasor:** Sem conseguir validar nomes de usuários ou versões específicas de serviços vulneráveis durante o reconhecimento, o custo operacional e a visibilidade do atacante aumentam consideravelmente, favorecendo a detecção precoce pela equipe de segurança.

---

### Medida 6: Monitoramento Contínuo, Correlação em SIEM e Resposta a Incidentes

#### A Abordagem Recomendada
Envio de todos os logs de autenticação (sucessos, falhas, bloqueios, criações de contas) para uma solução centralizada de SIEM/XDR com regras de correlação ativas.

#### Indicadores Críticos para Detecção:
1. Picos de falhas de autenticação com o mesmo código de erro originados de uma única subnet ou faixa de IPs.
2. Uma única senha sendo testada contra múltiplos usuários em intervalo curto de tempo (assinatura clássica de *Password Spraying*).
3. Logins ocorridos em horários atípicos ou provenientes de locais geograficamente impossíveis (*Impossible Travel*).

#### Como Auxilia na Proteção
* Permite à equipe de resposta a incidentes isolar endpoints comprometidos e banir endereços IP maliciosos na borda (Firewall/WAF) em tempo real, contendo a ameaça antes da fase de movimento lateral.

---

## 3. Matriz Comparativa de Defesas

| Medida de Segurança | Eficácia contra Força Bruta | Eficácia contra Password Spraying | Eficácia contra Enumeração | Impacto na Experiência do Usuário |
| :--- | :---: | :---: | :---: | :---: |
| **Passphrases Longas + Verificação de Vazamentos** | **Alta** | Média | Baixa | Baixo (fáceis de memorizar) |
| **MFA Obrigatório (FIDO2 / Number Matching)** | **Crítica** | **Crítica** | Baixa | Médio (rápida validação móvel) |
| **Expiração e Desativação de Contas Zumbis** | Média | **Alta** | Média | Baixo |
| **Smart Lockout & Rate Limiting** | **Alta** | **Alta** | Baixa | Baixo (não trava contas inocentes) |
| **Erros Genéricos & Ocultação de Banners** | Baixa | Baixa | **Alta** | Nulo |
| **Monitoramento Centralizado (SIEM/XDR)** | **Alta** | **Alta** | **Alta** | Nulo (camada interna do SOC) |

---

## 4. Conclusão

Nenhuma medida isolada é suficiente para barrar investidas modernas contra sistemas de autenticação. A segurança corporativa eficaz baseia-se no conceito de **Defesa em Profundidade (*Defense in Depth*)**: políticas de senhas robustas diminuem a previsibilidade, o MFA bloqueia tentativas baseadas em credenciais já expostas, o controle de enumeração dificulta o mapeamento do ambiente e o monitoramento garante visibilidade contra comportamentos anômalos. A combinação dessas camadas reduz significativamente o risco de violações e vazamentos de dados empresariais.
