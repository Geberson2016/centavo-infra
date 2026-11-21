# centavo-infra

Sobe o Centavo inteiro na máquina local com um comando:

```
Postgres → API Spring (centavo-backend) → BFF (centavo-bff) → UI (centavo-frontend, nginx)
```

Cada serviço é buildado a partir do `Dockerfile` do próprio repositório. Este repositório só tem o `docker-compose.yml` e o modelo de `.env`.

## Pré-requisitos

- Docker com Compose v2.
- Os repositórios clonados lado a lado:

```
Centavo/
  centavo-backend/
  centavo-bff/
  centavo-frontend/
  centavo-infra/     ← este
```

## Subir

```bash
cp .env.example .env    # e preencha (segredos: openssl rand -hex 32)
docker compose up -d --build --wait
```

Abra **http://localhost:4173**. O `--wait` só devolve o terminal quando todos os serviços estão saudáveis.

| Serviço  | No host                   | O que é                                          |
| -------- | ------------------------- | ------------------------------------------------ |
| web      | http://localhost:4173     | UI (nginx), com proxy de `/bff` para o BFF        |
| bff      | http://localhost:3000     | BFF; `GET /health` para checar                    |
| backend  | http://localhost:8090     | API Spring (`/api/v1/...`)                        |
| postgres | localhost:5433            | Banco (usuário e senha do `.env`)                 |

As portas ficam presas a `127.0.0.1` e podem ser trocadas no `.env` (`WEB_PORT`, `BFF_PORT`, `BACKEND_PORT`, `POSTGRES_PORT`).

## Dia a dia

```bash
docker compose logs -f bff           # logs de um serviço
docker compose up -d --build backend # rebuilda só o que mudou
docker compose down                  # para tudo (os dados do banco ficam no volume)
```

> `docker compose down -v` **apaga o volume do banco**. Só use se quiser começar do zero.

### Desenvolver com hot reload

Para mexer no BFF ou na UI, deixe no compose só o que não está editando e rode o resto com `npm run dev`:

```bash
docker compose up -d --wait postgres backend
# centavo-bff:       npm run dev   (BACKEND_URL=http://localhost:8090 no .env dele)
# centavo-frontend:  npm run dev   (http://localhost:5173, proxy de /bff para :3000)
```

## Notas

- No compose, o cookie de sessão sai **sem** a flag `Secure` (`SESSION_COOKIE_SECURE=false`), porque o stack local é HTTP. Em produção ela fica ligada.
- O volume `centavo-pgdata` tem o mesmo nome (e o mesmo projeto `centavo`) do compose antigo, então os dados de dev continuam. Ele foi criado com as credenciais do `.env`: mantenha as mesmas.
- Os testes não usam este banco: o backend sobe o próprio Postgres pelo Testcontainers.
