# ATLAS FOREX — etapa inicial executável

Plataforma independente de análise e configuração Forex. **Não envia ordens reais, não conecta à Exness e não mantém estratégia rodando ainda.** Essa separação é intencional: a documentação oficial pública consultada da Exness descreve Python integrado ao MT5 e automação em MT5; nesta pesquisa não foi localizado um endpoint oficial independente autorizado para conectar este app diretamente. Portanto, nenhum conector/credencial fictício foi criado.

## O que já funciona

- Interface web responsiva, navegação entre Terminal, Negociação manual, Central de automações, Backtesting, Histórico, Painel financeiro, Corretora e Configurações.
- Tema escuro/claro; PWA instalável com manifest e cache offline da interface.
- Lista dos dez pares pedidos e seletor de períodos M1–D1. Sem feed configurado, mostra indisponibilidade, não cotações de mentira.
- Central com 15 estratégias cadastráveis. Formulários salvam configurações localmente; botões explicam e bloqueiam ativação enquanto não houver motor/feed/corretora.
- Ferramenta local de backtest aceita CSV OHLC e exige declarar fonte. Implementada apenas para cruzamento SMA 9/21 (inclui resumo/curva, spread, slippage e comissão especificados). Outros tipos são bloqueados. Não calcula swap, não valida spread intrabar, não usa dados fornecidos por terceiros automaticamente e não constitui resultado de corretora.
- API Express, autenticação com hash de senha e JWT, SQLite, configuração de risco/auditoria. Rotas de ordens e início de automação falham fechadas; status relata sem conexão real.
- Testes de existência dos arquivos e bloqueios de segurança.

## Arquitetura

- `frontend/`: PWA (HTML/CSS/JS), UI responsiva, armazenamento das configurações no dispositivo.
- `backend/server.js`: API HTTP, autenticação, schema SQLite e rotas protegidas.
- `database/`: SQLite criado na inicialização (arquivo runtime ignorado pelo Git).
- `tests/`: testes Node nativos.
- `capacitor.config.json`: configuração inicial Capacitor para empacotar Android.

Tecnologias escolhidas: Node.js/Express para API simples de hospedar em VPS; SQLite no desenvolvimento inicial, com WAL e chaves estrangeiras; bcrypt para senha; JWT com segredo somente no backend; Zod para validar entradas; PWA para instalação sem loja. Para produção com múltiplos usuários, migrar para PostgreSQL e armazenamento de segredos KMS/secret manager.

## Requisitos

Node.js 20+, npm; para Android também Android Studio/SDK. O terminal usa Lightweight Charts carregado de CDN quando internet está disponível; os arquivos e interface base podem abrir offline, mas esse gráfico externo, dados de mercado, API e serviços de conta requerem rede. O backtest CSV é executado no navegador local e exige o arquivo selecionado.

## Executar localmente

```sh
cp .env.example .env
# Altere JWT_SECRET antes de expor o servidor.
npm install
npm test
npm start
```

Acesse `http://localhost:8080`. Para usar rotas autenticadas, registre e faça login em `/api/auth/register` e `/api/auth/login`, usando JSON `{"email":"voce@example.com","password":"uma-senha-com-12-ou-mais"}`. Envie o token como `Authorization: Bearer <token>`. Não publique o servidor em HTTP na internet: use HTTPS via proxy TLS, firewall e segredo aleatório forte. A autenticação é fundação inicial, não uma revisão de segurança completa.

## Backtest CSV

Formato com cabeçalho: `time,open,high,low,close`; opcional `spread`. Valores numéricos em preço e hora parseável. Na tela Backtesting, identifique a fonte real do arquivo. Testes SMA 9/21 abrem/fecham posições no cruzamento inverso, cobram spread/slippage configurados e comissão round-turn informada. Limitações atuais: o timeframe indicado não reamostra dados; datas não filtram o CSV; swap, gaps/slippage variável, conversão de moedas, margem, stop/take-profit, preço Bid/Ask por candle e regras específicas da corretora não são modelados. Não use como previsão nem evidência de rentabilidade. Para resultados profissionais, integrar série histórica licenciada e modelo auditado.

## Deploy / GitHub Pages

GitHub Pages serve estáticos, não Node/API, banco, autenticação server-side ou automações contínuas. Pode publicar `frontend/` (em Settings > Pages, branch/pasta apropriada), preferencialmente com HTTPS. Como a API ainda não está hospedada e a UI foi escrita para ser servida junto do backend, publicação Pages é somente UI offline/estática; não habilita conta ou operação. Não incluir `.env`, banco, senha ou token no repositório. Hospede o backend em VPS/serviço com HTTPS, disco persistente, backup e CORS configurado antes de integrar.

## PWA e Android

Servir sob HTTPS (ou localhost para desenvolvimento) e usar opção “Instalar aplicativo” do navegador Android. Offline: shell HTML/CSS/JS/manifest/ícone em cache e configurações no armazenamento local. Online obrigatório: conteúdo CDN do gráfico, autenticação, feed e qualquer serviço futuro. A interface não faz automação em background.

Para iniciar empacotamento Android, instale Android Studio e SDK, depois:

```sh
npm install
npx cap add android
npx cap sync android
npx cap open android
```

No Android Studio, gere APK debug para teste; APK assinado de distribuição exige keystore própria e revisão. Este ambiente não possui Android SDK/Gradle instalado, então nenhum APK foi gerado/verificado.

## Exness, segurança e próximos bloqueios

Fonte consultada: [artigo oficial Exness sobre Python e MT5](https://www.exness.com/blog/behind-the-markets/an-introduction-to-algorithmic-trading-with-python-and-mt5) e [guia oficial de automação Exness](https://www.exness.com/indices/algorithmic-trading/). Os artigos ilustram automação com MT5; não comprovam autorização de API direta para este frontend/backend. Confirme com a Exness os meios e termos atuais aplicáveis à sua conta/região antes de qualquer execução. Nenhuma senha deve ser enviada no chat, commitada ou colocada no cliente.

Para operação contínua real faltam: contrato/feed de dados autorizado, adaptador de execução permitido (ou protocolo oficial confirmado), infraestrutura VPS, segredos em vault, semântica de reconciliação/ordens idempotentes, gerenciamento de estado após restart, proteção de perda diária/total testada, auditoria independente, testes de integração/paper-trading e verificação regulatória/jurídica. A rota de start e ordens está deliberadamente bloqueada até estes requisitos.
