# Deploy — guia geral do portfólio

A infraestrutura de deploy é **a mesma para todos os projetos**: uma VM Oracle Cloud
(Always Free) rodando Docker, com Caddy fazendo HTTPS automático. Só mudam alguns detalhes
por linguagem (porta interna, variáveis de produção).

## A base (idêntica para todos)

O passo a passo completo — criar a conta Oracle, gerar a chave SSH, criar a VM ARM, abrir
o firewall (o duplo firewall!), instalar Docker, configurar DuckDNS e subir com Caddy —
está detalhado no **[`docs/DEPLOY.md` do DocSage](https://github.com/Everett-gi/docsage/blob/main/docs/DEPLOY.md)**
(que tem repositório próprio). Ele serve para qualquer projeto deste portfólio; apenas
troque o repositório, a pasta e o domínio.

Resumo do fluxo para os projetos deste monorepo, uma vez que a VM já existe:

```bash
ssh docsage                                   # conecta na VM
git clone https://github.com/Everett-gi/Projetos-e-ideias.git
cd Projetos-e-ideias/python/<projeto>        # monorepo: entre na pasta do projeto
nano .env                                      # .env de PRODUÇÃO, criado aqui (não vem do Git)
chmod 600 .env
docker compose -f docker-compose.prod.yml up -d --build
```

O DocSage é a exceção: clone `https://github.com/Everett-gi/docsage.git` e rode da raiz
dele (`cd docsage`), como descreve o DEPLOY.md dele.

## Rodando vários projetos na mesma VM

A VM Always Free (2 OCPU / 12 GB ARM) aguenta poucos serviços vivos ao mesmo tempo —
principalmente os de Java, que consomem mais memória. Estratégia recomendada:

- Mantenha **3 a 5 projetos vivos** (os carros-chefe) e os demais prontos para subir.
- **Um Caddy só** para a VM toda, com um bloco por domínio, roteando para cada aplicação.
  Assim você não sobe um Caddy por projeto.
- Cada projeto usa seu próprio subdomínio DuckDNS (ex.: `authhub-gil.duckdns.org`).
- Se faltar memória, limite o heap dos apps Java (`-XX:MaxRAMPercentage=50`) e use swap.

Exemplo de um Caddyfile central na VM (fora dos projetos), com vários apps:

```
authhub-gil.duckdns.org   { reverse_proxy authhub-app:8080 }
docsage-gil.duckdns.org   { reverse_proxy docsage-app:8080 }
mercado-gil.duckdns.org   { reverse_proxy mercadolite-app:3000 }
```

## Notas por linguagem

### Java (Spring Boot)
- Porta interna: **8080**.
- Build multi-stage (Maven → JRE), imagem final enxuta, usuário não-root.
- Variáveis de produção típicas: `SPRING_PROFILES_ACTIVE=prod`, `SPRING_DATASOURCE_URL`,
  `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`, `JWT_SECRET`.
- Memória: JVM respeita o container com `-XX:MaxRAMPercentage`. Comece com 50–75%.
- Migrations com Flyway rodam na subida da aplicação.

### Python (FastAPI)
- Porta interna: **8080** (uvicorn).
- Detalhes completos no [`docs/DEPLOY.md` do DocSage](https://github.com/Everett-gi/docsage/blob/main/docs/DEPLOY.md) (é o modelo).
- Projetos com modelo de ML/embeddings: a 1ª build é lenta (baixa PyTorch) — normal.
- Migrations com Alembic: rode `alembic upgrade head` na subida (entrypoint ou comando).

### Ruby (Rails)
- Porta interna: **3000**.
- Exige `RAILS_MASTER_KEY` (do `config/master.key`, que **não** vai pro Git) no `.env`
  de produção, e `SECRET_KEY_BASE`.
- Rode as migrations e o precompile de assets no deploy:
  `bin/rails db:migrate && bin/rails assets:precompile`.
- Use `RAILS_ENV=production` e `RAILS_SERVE_STATIC_FILES=true` (Caddy também pode servir).

## Checklist de deploy (qualquer projeto)

- [ ] Repositório no GitHub, com Secret Scanning + Push Protection ativos
- [ ] `.env` de produção criado **na VM** (nunca versionado), com `chmod 600`
- [ ] Subdomínio DuckDNS apontando para o IP da VM (confirmado com `dig`)
- [ ] Portas 80/443 abertas na Security List **e** no firewall do sistema (iptables)
- [ ] `docker compose -f docker-compose.prod.yml up -d --build` sem erros
- [ ] HTTPS funcionando (cadeado no navegador, sem aviso)
- [ ] Backup do banco agendado (cron) e, idealmente, enviado para storage externo
