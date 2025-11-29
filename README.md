# Newsletter Financeiro Automatizada (AI Powered)

Gerador diário de newsletter financeira em português (Brasil) usando um agente de IA com pesquisa web (Tavily) e envio automático por e‑mail. Ele:

- Pesquisa múltiplas fontes confiáveis (nacionais e internacionais)
- Organiza manchetes principais por região (Brasil, Europa, EUA)
- Resume movimentos de mercado: ações, câmbio, commodities e macro
- Formata em um template padronizado pronto para envio
- Envia automaticamente no horário configurado via SMTP (Gmail)

> Objetivo: agilizar a produção de um Morning Call alternativo confiável, estruturado e escalável para investidores, gestores e analistas.

---
## 1. Visão Geral
O núcleo do projeto é um agente (`agno.Agent`) que combina:

1. Modelo OpenAI (`gpt-4.1-mini` – configurável)
2. Ferramenta de busca Tavily (para encontrar notícias e dados)
3. Função interna de envio de e‑mail (`envia_email_tool`)
4. Prompt robusto (em `prompt.py`) que define estrutura, estilo e validação.

O script principal (`03.news_financeira.py`) gerencia o ciclo diário: executa o agente, gera a newsletter e agenda novo envio conforme a variável de ambiente `SEND_AT`.

---
## 2. Arquitetura
```
prompt.py (template e regras da newsletter)
01.agente.py (exemplo simples de agente pesquisando oportunidades)
02.email_tool.py (função isolada para teste de envio de email)
03.news_financeira.py (rotina principal: agent + agendamento + envio)
requirements.txt (dependências do ambiente)
.env (não incluso no repo – contém chaves/segredos)
```

Fluxo resumido:
```
Carrega variáveis .env
↓
Instancia Agent (modelo + Tavily + função de email)
↓
Executa prompt base (gera edição do dia)
↓
Loop aguardando horário HORA_ENVIO (SEND_AT)
↓
Refaz prompt com DATA atual → envia email
```

---
## 3. Dependências e Requisitos
Python 3.10+ recomendado.

Arquivo `requirements.txt`:
```
agno==2.1.0
openai>=1.107.1
python-dotenv>=1.1.1
tavily-python>=0.7.12
```

Você também precisa das chaves de API:
- `OPENAI_API_KEY`
- `TAVILY_API_KEY`

E credenciais de e‑mail (Gmail ou outro SMTP compatível).

---
## 4. Configuração do Ambiente
Crie um arquivo `.env` na mesma pasta dos scripts com, por exemplo:
```
OPENAI_API_KEY=sk-xxxxxxxxxxxxxxxx
TAVILY_API_KEY=tvly_xxxxxxxxxxxxxx
EMAIL_ADDRESS=seu_email@gmail.com
EMAIL_PASSWORD=senha_ou_app_password
DESTINATARIOS=dest1@exemplo.com,dest2@exemplo.com
SEND_AT=08:30   # Formato HH:MM (BRT)
```

Para Gmail com autenticação de dois fatores, crie um App Password em vez da senha normal.

---
## 5. Instalação
Crie e ative um ambiente virtual (opcional, recomendado):
```powershell
python -m venv .venv
.\.venv\Scripts\activate
pip install -r requirements.txt
```

Teste rápido das libs:
```powershell
python -c "import agno, openai, tavily; print('OK')"
```

---
## 6. Execução
### 6.1. Execução imediata (gerar uma edição só)
```powershell
python 03.news_financeira.py
```
O script rodará o agente e imprimirá o agendamento. A newsletter será reenviada automaticamente no horário definido em `SEND_AT`.

### 6.2. Teste do envio de e‑mail isolado
```powershell
python 02.email_tool.py
```

### 6.3. Ajustar o prompt
Edite `prompt.py` para mudar seções, tom, fontes, requisitos e estrutura.

---
## 7. Personalização
| Item | Como alterar |
|------|--------------|
| Modelo | Mudar `model=OpenAIChat(id="gpt-4.1-mini")` para outro ID suportado |
| Fontes | Editar lista de fontes no `prompt.py` |
| Horário de envio | Variável `SEND_AT` no `.env` |
| Idioma | Ajustar instruções no prompt (pt-BR → en-US, etc.) |
| Template | Modificar bloco FORMATO DE SAÍDA em `prompt.py` |
| Ferramentas | Adicionar outras funções na lista `tools=[...]` |

---
## 8. Segurança e Boas Práticas
- Nunca commitar `.env` ou chaves direto no repositório.
- Limitar exposição de senhas: usar App Password para Gmail.
- Validar sempre dados numéricos gerados (IA pode errar valores de mercado).
- Adicionar logs e/ou persistência se for usar em produção (ex.: gravar newsletters geradas em `logs/` ou banco).
- Considerar filtro anti‑alucinação: pós-processar números contra fonte oficial.

---
## 9. Estrutura do Prompt (Qualidade)
O `prompt.py` faz engenharia de prompt avançada:
- Define papel (economista sênior)
- Impõe formato estável
- Regras de validação (mínimo de fontes distintas, horários BRT, evitar placeholders)
- Gatilho para função de envio (assunto padronizado)

Recomendação: manter seções curtas, numéricas verificáveis e links completos.

---
## 10. Extensões / Roadmap Sugerido
- [ ] Cache de resultados (evitar repetir buscas iguais no dia)
- [ ] Persistência em banco (SQLite / Postgres)
- [ ] Painel web para revisar antes do envio
- [ ] Multi‑canal (Telegram / Slack / WhatsApp API)
- [ ] Métrica de cobertura (quantidade de fontes, diversidade)
- [ ] Testes automáticos para formato final (regex de validação)
- [ ] Suporte a outras línguas

---
## 11. Limitações
- Dependente da qualidade das ferramentas externas (Tavily / OpenAI)
- Possível latência em mornings com grande volume de notícias
- IA pode produzir resumos com nuance perdida; revisar tópicos sensíveis
- Números precisam conferência manual antes de uso institucional

---
## 12. Logs e Debug
`debug_mode=True` no agente imprime passos internos (útil para auditoria). Para produção, considere desligar para reduzir custo e ruído.

---
## 13. Testes Manuais Rápidos
1. Validar envio de email → `python 02.email_tool.py`
2. Rodar agente simples → `python 01.agente.py`
3. Forçar geração única (sem loop) → comentar o `while True` em `03.news_financeira.py`

---
## 14. Perguntas Frequentes (FAQ)
**P: Posso usar outro provedor de e‑mail?** Sim, ajuste host/porta SMTP.
**P: Como mudar a frequência?** Alterar lógica do loop (ex: agendar várias horas). 
**P: Posso gerar PDF?** Após texto, usar biblioteca `reportlab` ou `weasyprint`.
**P: Evitar repetição de fontes?** Já há regra no prompt; pode adicionar checagem pós-processamento.

---
## 15. Contribuição
1. Abra issue descrevendo melhoria
2. Faça fork / branch
3. Mantenha estilo simples, sem comentários desnecessários
4. Pull request com descrição clara (ANTES / DEPOIS)

---
## 16. Licença
Não definida neste repositório. Adicione uma licença (MIT / Apache 2.0 / Proprietária) conforme necessidade.

---
## 17. Aviso Legal
Conteúdo gerado por IA pode conter imprecisões. Não constitui recomendação de investimento. Verifique dados críticos em fontes oficiais (B3, Banco Central, CVM, Relatórios Corporativos).

---
## 18. Exemplo de Assunto e Output
Assunto: `Newsletter Financeiro AI - 29/11/2025`
Corpo: Estrutura conforme bloco padrão (ver `prompt.py`).

---
## 19. Próximos Passos Sugeridos
- Integrar verificação automática de números (ex.: API B3 / Banco Central)
- Criar testes unitários para função de envio
- Adicionar parâmetro para desabilitar emojis

---
## 20. Suporte
Para dúvidas técnicas: abra issue ou ajuste diretamente seu fork. Para integrações corporativas, considere wrappers adicionais (logging robusto, autenticação, fila de tarefas).

---
### Fim
"🤖 Powered by Inteligência Artificial" — mantenha esta linha para rastreabilidade.

