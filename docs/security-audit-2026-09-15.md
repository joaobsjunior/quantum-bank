# Quantum Bank — Auditoria de Segurança (2026-09-15)

Escopo: superprojeto e todos os submódulos (`backend`, `backend-client`,
`api-gateway`, `infrastructure`, `mobile-app`, `pki`) na versão mais recente de
`main`. Método: leitura completa do código-fonte, configurações, scripts e
workflows, seguida de correção e verificação executável (testes unitários com
gate de 100% de cobertura, lint do KrakenD e execução ponta a ponta do stack
Docker Compose com smoke e testes negativos de mTLS).

## Resumo

| Severidade | Encontradas | Corrigidas |
| --- | --- | --- |
| Alta | 5 | 5 |
| Média | 9 | 9 |
| Baixa | 8 | 8 |
| Informativa (limitações de arquitetura v1) | 4 | documentadas |

## Achados e correções

### Alta

1. **Injeção de configuração OpenSSL via identificadores do cliente**
   (`pki/scripts/sign-csr.sh`, `backend/.../PkiAdapter.kt`). `appInstanceId`,
   `deviceId` e o `sub` eram escritos sem validação no arquivo de extensões que
   alimenta `openssl x509`. Uma quebra de linha permitia injetar
   `basicConstraints=CA:TRUE`, `extendedKeyUsage=serverAuth` ou SANs de
   `gateway-banking`, ou seja, emitir certificados arbitrários assinados pela CA
   local. Correção: charset canônico obrigatório em três camadas — validação
   Bean Validation nos DTOs HTTP (`BootstrapIdentifiers`), revalidação antes do
   `ProcessBuilder` e validação com regex dentro do próprio script.
2. **Identidade colapsada por `azp`/`preferred_username`** (`JwtSubject.kt`,
   realm Keycloak). O backend aceitava o client id como sujeito quando `sub`
   faltava; como o realm importado não incluía o client scope `basic`, todos os
   usuários do cliente `quantum-bank-test` compartilhavam o mesmo perfil e
   extrato. Correção: apenas `sub` é aceito (401 `auth_invalid_token` caso
   contrário); o realm ganhou o scope `basic` com `oidc-sub-mapper` em todos os
   clientes; contas de serviço têm ids fixos e fixtures em `data.sql`.
3. **Console do serviço externo sem autenticação nem CSRF** (`backend-client`).
   Qualquer host que alcançasse a porta 8090 disparava Pix com as credenciais
   OAuth2 e o certificado mTLS do serviço. Correção: Spring Security com login
   de operador (credenciais obrigatórias por variável de ambiente, senha
   mínima de 12 caracteres), CSRF em todos os POSTs, CSP e cabeçalhos padrão,
   imagem Docker sem root. Verificado por curl: anônimo → 302 `/login`, POST
   sem token → 403, senha errada → `/login?error`, fluxo autenticado → 200.
4. **Emissor OAuth2 em texto claro e `disable_jwk_security: true`**
   (`compose.yaml`, KrakenD, backend, backend-client, mobile). Tokens, senha do
   grant ROPC, segredo do client e JWKs trafegavam em HTTP. Correção: Keycloak
   em modo produção somente HTTPS (`KC_HTTP_ENABLED=false`) com certificado
   emitido pela PKI local; KrakenD usa `jwk_local_ca`; backend confia apenas na
   truststore da PKI para buscar o JWK e recusa emissor não HTTPS fora de
   `local`; backend-client exige `https` para token e gateway; app mobile
   recusa qualquer origem `http` e fixa a raiz local como única âncora do
   cliente Keycloak.
5. **Testes negativos de mTLS passavam vaziamente** (`negative-mtls-tests.sh`).
   Os fixtures referenciados (`local-ca/negative/*`) nunca eram gerados e a
   mensagem "could not load PEM client certificate" casava com o padrão
   `certificate`, aprovando todos os casos. Correção: fixtures reais (CA não
   confiável, certificado expirado, CA de outro ambiente) gerados no bootstrap,
   pré-checagem de existência, padrão de erro restrito à camada TLS, serviço
   Compose dedicado e execução real no e2e.

### Média

6. **Sem prova de posse do CSR** — a assinatura PKCS#10 não era verificada
   antes de consumir o OTK. Correção: `isSignatureValid` (BouncyCastle) no
   `CsrValidator` e `openssl req -verify` no script.
7. **Sem política de chave** — CSRs com RSA-1024 ou curvas fracas eram
   assinados. Correção: RSA ≥ 2048 ou EC ≥ 256 bits (`csr_key_rejected`) no
   backend e no script.
8. **Vínculo de sujeito por substring** — `subject.contains(sub)`. Correção:
   exatamente um CN, igual byte a byte, no backend e no script.
9. **Bypass do gateway com qualquer certificado da CA** — o backend aceitava
   qualquer certificado cliente da PKI (mobile, serviço externo). Correção:
   `GatewayClientCertificateFilter` restringe o CN ao `gateway-client`
   (403 `mtls_client_not_allowed`), habilitado no Compose e coberto no smoke.
10. **Rotas não mapeadas autenticáveis só por mTLS** — `anyRequest().authenticated()`.
    Correção: `denyAll()` com teste.
11. **Repositório de OTK ilimitado** — crescimento de memória sem purga e sem
    limite por sujeito. Correção: purga por retenção, um OTK ativo por
    sujeito/app/dispositivo (anterior revogado) e teto configurável com falha
    fechada (503 `otk_capacity_exceeded`).
12. **Segredos e credenciais com fallback embutido** — `change-me-local-only`
    no `application.yml` do backend-client, no realm versionado e no binário
    Flutter. Correção: segredo do client e credenciais do console obrigatórios
    (startup falha sem eles); realm recebe segredos e senha de teste por
    substituição de ambiente na importação; app mobile sem senha padrão
    (`--dart-define=KEYCLOAK_PASSWORD`).
13. **Sem limitação de taxa nem timeouts no gateway** — OTK/CSR (que disparam
    processos OpenSSL) podiam ser inundados. Correção: `qos/ratelimit/router`
    por IP em todos os endpoints, timeouts de leitura/cabeçalho/ocioso, TLS 1.3
    mínimo, `security/http` (HSTS, nosniff, frame-deny, referrer policy).
14. **CA local inconsistente após clone** — `bootstrap-local-ca.sh` mantinha o
    certificado versionado mesmo quando a chave local não correspondia,
    quebrando toda assinatura silenciosamente. Correção: bootstrap
    auto-corretivo (reemite âncoras quando chave ≠ certificado) e
    `verify-trust-anchors.sh` valida correspondência chave/certificado e
    `CA:TRUE`.

### Baixa

15. Correlation id refletido sem validação em cabeçalho, banco e logs →
    formato restrito, substituído por UUID quando inválido.
16. Campos Pix/perfil sem limite → estouro de coluna virava 500 →
    `@Size`/`@Digits`/`@DecimalMax` com 400 `request_invalid`.
17. Saída do processo de assinatura lida após `waitFor` → possível bloqueio
    por buffer → redirecionada para arquivo; validade do certificado lida do
    próprio X.509.
18. Serial da CA sem exclusão mútua → `flock` e arquivo de estado fora da
    árvore versionada (permite montar `trust/` como somente leitura).
19. Portas do Compose expostas em todas as interfaces → `127.0.0.1` por padrão
    (`BIND_ADDRESS`).
20. Container do backend com o repositório PKI inteiro montado em leitura e
    escrita → montagens mínimas e somente leitura (scripts, perfis, chave
    privada, âncoras).
21. `webOrigins: "+"` e redirect de logout curinga no cliente público →
    removidos; `sslRequired: all`, brute-force protection e lifespan de 5 min
    no realm.
22. BouncyCastle 1.80 → 1.86; TLS do backend-client restrito a 1.2/1.3 com
    timeout de conexão; smoke e job e2e do CI corrigidos para gerar PKI,
    usar `--env-file` e executar os dois conjuntos de testes.

### Informativas (não corrigíveis sem mudança de arquitetura v1)

- O backend executa o script de assinatura e, portanto, tem acesso de leitura
  à chave da CA emissora. Recomenda-se um serviço de PKI isolado (por exemplo
  OpenXPKI) antes de qualquer ambiente compartilhado.
- KrakenD aceita qualquer certificado cliente emitido pela raiz; não há
  vínculo entre o certificado apresentado e o `sub` do JWT nem verificação de
  revogação (CRL/OCSP). Mitigado pela validade de 1 dia dos certificados
  mobile e pelo guard de identidade no backend.
- O app mobile usa o grant ROPC (senha) e mantém chave privada e certificado
  em memória; migrar para Authorization Code + PKCE e armazenamento seguro do
  sistema operacional.
- Imagens Terraform apontam para tags `latest`; fixar por digest antes de usar
  em nuvem.

## Verificação executada

| Camada | Resultado |
| --- | --- |
| backend `gradle check` | 81 testes, Kover 100 % (895 linhas) |
| backend-client `gradle check` | testes verdes, Kover 100 % (211 linhas) |
| mobile `flutter test --coverage` | 566/566 linhas, `coverage-ok` |
| api-gateway `krakend check --lint` | 3 configs `Syntax OK` |
| pki `verify-trust-anchors.sh`, matriz do `sign-csr.sh` | ok; chaves fracas, CN errado, assinatura inválida e injeção rejeitados |
| infrastructure e2e (`smoke-tests`, `negative-mtls-tests`) | `local-e2e-smoke-ok`, `negative-mtls-ok` |
| console backend-client via curl | login, CSRF e fluxo de extrato (HTTP 200) |

## Ações pendentes para o mantenedor

- Copiar `infrastructure/.env.example` para `.env` e trocar todos os valores
  `change-me-local-only`/`changeit`; definir `PKI_GID` com o gid dono de
  `pki/local-ca/private`.
- As âncoras `pki/local-ca/trust/*.crt` e `mobile-app/assets/local-ca/root-ca.crt`
  foram reemitidas nesta máquina (a chave local não correspondia ao certificado
  versionado); versionar as novas âncoras junto com as demais mudanças.
- Regerar os artefatos com `pki/scripts/bootstrap-runtime-certs.sh` em cada
  máquina antes de subir o stack.
