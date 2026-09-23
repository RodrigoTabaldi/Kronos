# Kronos

Plataforma web de gestão financeira e redução de endividamento para pessoas físicas, autônomos/MEI e empresas.

O Kronos combina lançamentos financeiros, análise de dívidas, simuladores, relatórios e recomendações calculadas localmente no navegador. O estado da conta é autenticado e sincronizado pelo Firebase.

> Status: MVP em desenvolvimento.

## Visão rápida

- **Frontend:** React 19, TypeScript strict, Vite 6 e Tailwind CSS.
- **Navegação:** React Router; as rotas internas exigem login.
- **Autenticação:** Firebase Authentication com e-mail e senha.
- **Persistência:** um snapshot JSON por usuário no Firestore, em `kronos_snapshots/{uid}`.
- **Cálculos:** TypeScript no cliente, sem API própria ou banco relacional no runtime atual.
- **Testes:** Vitest, com foco atual no motor de dívidas.

## Funcionalidades

### Módulos principais

| Rota | O que oferece |
| --- | --- |
| `/cadastro` e `/login` | Criação e acesso da conta com perfil PF, PJ/MEI, pequena empresa ou empresa. |
| `/dashboard` | Indicadores, fluxo de caixa, receitas, despesas, categorias, alertas e resumo de dívidas. |
| `/copiloto` | Kairós, um conjunto de regras locais que prioriza a próxima ação financeira. |
| `/dividas` | Cadastro de dívidas, índice de saúde, comprometimento, Avalanche x Bola de Neve, jornada de quitação, trocas de crédito e renegociação. |
| `/transacoes` | Inclusão, edição, exclusão, filtros, ordenação e status pago/pendente. |
| `/notas-fiscais` | Emissão de recibos, visualização e impressão/PDF pelo navegador. |
| `/viabilidade` | Break-even, margem, cenários e metas de faturamento. |
| `/impostos` | Calculadoras de MEI, Simples, IRPF e IRRF, além de vencimentos registrados. |
| `/investimentos` | Juros compostos, comparação de aplicações, financiamento, inflação e consórcio. |
| `/relatorios` | DRE simplificada, fluxo de caixa, detalhamento mensal, exportação CSV e impressão/PDF. |
| `/settings` | Dados do negócio, documento, regime, plano, logo e encerramento da sessão. |

### Análises avançadas

Essas rotas são carregadas sob demanda para reduzir o carregamento inicial:

| Rota | Análise |
| --- | --- |
| `/analises/previsao` | Projeção de fluxo para até 90 dias, com tendência e intervalo estatístico. |
| `/analises/benchmarking` | Comparação com referências estáticas do segmento. |
| `/analises/estilo-vida` | Despesas pessoais do dono versus lucro do negócio. |
| `/analises/capital-giro` | Ciclo de caixa, reserva necessária e soluções. |
| `/analises/detectar-fuga` | Anomalias e economia potencial nas despesas. |
| `/analises/otimizar-preco` | Cenários de preço, demanda e lucro. |
| `/analises/mix-servicos` | Pareto 80/20 e classificação A/B/C dos serviços. |
| `/analises/roi` | Payback, ROI, VPL, TIR e sensibilidade. |
| `/analises/variance` | Decomposição da mudança do lucro entre períodos. |
| `/analises/decisao` | Comparação de opções por risco e retorno. |
| `/analises/vieses` | Sinais de vieses cognitivos identificados nos dados. |
| `/analises/simulador` | Cenários estratégicos salvos e comparados lado a lado. |

## Pré-requisitos

- Node.js 20 ou superior. O CI do projeto usa Node.js 20.
- npm.
- Um projeto Firebase para executar o app com login e persistência.

## Instalação e execução local

1. Instale as dependências:

   ```bash
   npm install
   ```

2. Crie o arquivo de ambiente.

   No PowerShell:

   ```powershell
   Copy-Item .env.example .env
   ```

   No macOS/Linux:

   ```bash
   cp .env.example .env
   ```

3. Preencha o `.env` com a configuração do seu app web no Firebase. Os nomes das variáveis estão em [`.env.example`](./.env.example).

4. Inicie o servidor:

   ```bash
   npm run dev
   ```

   Abra o endereço mostrado pelo Vite, normalmente `http://localhost:5173`.

Sem as variáveis mínimas do Firebase, o app não consegue inicializar a camada de autenticação e Firestore no navegador.

## Configuração do Firebase

No [Firebase Console](https://console.firebase.google.com/):

1. Crie ou selecione um projeto.
2. Em **Authentication > Sign-in method**, habilite **E-mail/senha**.
3. Crie o **Firestore Database**.
4. Copie a configuração do seu app web para o `.env`:

   | Variável | Origem |
   | --- | --- |
   | `VITE_FIREBASE_API_KEY` | Configuração do SDK web |
   | `VITE_FIREBASE_AUTH_DOMAIN` | Configuração do SDK web |
   | `VITE_FIREBASE_PROJECT_ID` | Configuração do SDK web |
   | `VITE_FIREBASE_STORAGE_BUCKET` | Configuração do SDK web |
   | `VITE_FIREBASE_MESSAGING_SENDER_ID` | Configuração do SDK web |
   | `VITE_FIREBASE_APP_ID` | Configuração do SDK web |

5. Publique as regras versionadas antes de usar dados reais:

   ```bash
   npm install -g firebase-tools
   firebase login
   firebase deploy --only firestore:rules --project SEU_PROJETO
   ```

As regras em [`firestore.rules`](./firestore.rules) permitem acesso somente quando o `uid` autenticado é igual ao identificador do documento. Não use o Firestore em modo de teste para dados reais.

## Como os dados funcionam

```text
Firebase Auth ──> sessão do usuário
                      │
                      v
Interface React ──> StoreProvider/useReducer ──> Firestore
                              ^                    │
                              └── listener em tempo real
```

- O login e o cadastro usam Firebase Authentication.
- O estado financeiro é mantido no `StoreProvider`.
- Após a hidratação, alterações são salvas com debounce de aproximadamente 800 ms.
- Alterações recebidas de outro aparelho chegam pelo listener do Firestore.
- Uma conta nova começa sem transações, dívidas ou outros dados financeiros. Apenas categorias genéricas são criadas para os formulários.
- O arquivo [`supabase/schema.sql`](./supabase/schema.sql) é uma referência de modelagem relacional para uma possível migração futura; ele não é usado pelo app atual e não precisa ser executado para rodar o projeto.

## Verificação

```bash
# TypeScript + bundle de produção
npm run build

# Testes uma vez
npm test

# Testes em modo observação
npm run test:watch
```

O workflow [`ci.yml`](./.github/workflows/ci.yml) executa `npm ci`, type-check, testes e build a cada push em `main` e a cada pull request.

Os testes atuais cobrem principalmente [`lib/divida.ts`](./lib/divida.ts): juros, pagamento mínimo, sistema Price, saúde financeira, Avalanche, Bola de Neve, renegociação e casos de borda.

## Build e publicação

```bash
npm run build      # gera dist/
npm run preview    # serve o build localmente
```

Para Vercel ou Netlify:

1. Configure as seis variáveis `VITE_FIREBASE_*` no ambiente de build.
2. Use `npm run build` como comando de build.
3. Publique a pasta `dist`.
4. Mantenha o fallback de SPA configurado. O repositório já contém [`netlify.toml`](./netlify.toml) e [`vercel.json`](./vercel.json) para isso.
5. Publique também as regras do Firestore no projeto Firebase correto.

## Estrutura do projeto

```text
src/
  main.tsx                 entrada React e BrowserRouter
  App.tsx                  rotas públicas, protegidas e lazy loading
  pages/                   telas principais e análises avançadas
components/                shell, autenticação, gráficos, UI e animações
lib/
  types.ts                 tipos de domínio
  store.tsx                estado global e persistência acionada pela UI
  cloud.ts                 autenticação e Firestore
  firebase.ts              inicialização do Firebase
  divida.ts                motor de desendividamento
  jornada.ts               marcos e conquistas da quitação
  copiloto.ts              recomendações locais do Kairós
  utils.ts                 cálculos financeiros e fiscais
  format.ts                formatação e utilitários de apresentação
  seed.ts                  categorias iniciais
  divida.test.ts           testes do motor de dívidas
components/ui/             componentes visuais reutilizáveis
public/                    arquivos públicos estáticos
supabase/schema.sql        referência de schema PostgreSQL, não runtime
firestore.rules            regras de segurança do Firestore
.github/workflows/ci.yml   validação automatizada
```

## Limitações conhecidas do MVP

- A integração com Claude/IA é apenas prevista. O Kairós e as análises usam regras e cálculos locais.
- O benchmarking usa referências estáticas; ainda não existe uma base anônima real de usuários.
- As tabelas fiscais estão fixadas no código como referência de MVP, incluindo valores de 2024/2025. Confirme a legislação vigente antes de tomar decisões fiscais.
- O recibo pode ser impresso ou salvo como PDF pelo diálogo do navegador. Isso não equivale à emissão de uma NFS-e municipal nem a uma assinatura digital com certificado.
- Não há integração externa com bancos, adquirentes, contabilidade, pagamentos ou emissão fiscal oficial.
- O Firestore armazena um documento JSON por usuário. Isso simplifica o MVP, mas limita consultas relacionais e relatórios mais complexos.
- A suíte automatizada ainda não cobre a integração real com Firebase, as regras do Firestore, autenticação em produção ou publicação nos provedores.

## Segurança e dados sensíveis

- Não commit o arquivo `.env`.
- As chaves `VITE_FIREBASE_*` identificam o app web, mas não substituem as regras de segurança.
- O isolamento entre usuários depende das regras publicadas em [`firestore.rules`](./firestore.rules).
- Valide as regras com um projeto de teste ou o Firebase Emulator antes de publicar mudanças.
