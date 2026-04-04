# Installing OpenClaw with one command (Supports multiple OpenClaw instances on one server).

## I. Quick Installation

Install [docker](https://docs.docker.com/get-docker/) and [docker-compose](https://docs.docker.com/compose/install/) yourself, then run the following to set up an OpenClaw instance @ `localhost:18789` (default gateway password: `minhng.info`):

```bash
curl -fsSL https://raw.githubusercontent.com/minhng92/openclaw-minimal-docker-compose/main/run.sh | bash -s -- --destination openclaw-one
```

And/or run the following to set up another OpenClaw instance @ `localhost:18889`:

```bash
curl -fsSL https://raw.githubusercontent.com/minhng92/openclaw-minimal-docker-compose/main/run.sh | bash -s -- --destination openclaw-two --port 18889 --port-ws 18890
```

Arguments:
* `--destination`: Install folder (e.g., **openclaw-one**)
* `--port`: Gateway HTTP port (e.g., **18789**)
* `--port-ws`: Gateway WebSocket port (e.g., **18790**)
* `--password`: Gateway login password. Optional, defaults to `minhng.info`.
* `--openai-api-key`: OpenAI API key (written to **agent.env**). Optional.
* `--telegram-bot-token`: Telegram bot token (written to **agent.env**). Optional.

Example with a custom gateway password:

```bash
curl -fsSL https://raw.githubusercontent.com/minhng92/openclaw-minimal-docker-compose/main/run.sh | bash -s -- --destination openclaw-one --password mySecret123
```

Example with a gateway password + OpenAI key:

```bash
curl -fsSL https://raw.githubusercontent.com/minhng92/openclaw-minimal-docker-compose/main/run.sh | bash -s -- --destination openclaw-one --password mySecret123 --openai-api-key sk-proj-abc123XYZ
```

Example with all options (custom ports + password + OpenAI + Telegram):

```bash
curl -fsSL https://raw.githubusercontent.com/minhng92/openclaw-minimal-docker-compose/main/run.sh | bash -s -- \
  --destination openclaw-one \
  --port 18789 \
  --port-ws 18790 \
  --password mySecret123 \
  --openai-api-key sk-proj-abc123XYZ \
  --telegram-bot-token XXX:XXX
```

If `curl` is not found, install it:

```bash
sudo apt-get install curl
# or
sudo yum install curl
```

## II. Manual Installation

```bash
git clone git@github.com:minhng92/openclaw-minimal-docker-compose.git
cd openclaw-minimal-docker-compose
# cp .env agent.env
# nano agent.env        # configure domain, model provider, telegram
docker-compose up -d
```

`agent.env` is for deployment:
- Overrides the default `.env`
- Ignored by `.gitignore`

If you want to start the server with a different port, change 18789 and 18790 (before colon character) to another value in docker-compose.yml:

```yml
    ports:
      - "18789:18789"
      - "18790:18790"
```

### Paring on the first connection

Open `localhost:18789` and login with password (default: `minhng.info`). It shows _paring required_.

We have to approve the pending request to access OpenClaw Control UI: [https://docs.openclaw.ai/web/control-ui#device-pairing-first-connection](https://docs.openclaw.ai/web/control-ui#device-pairing-first-connection)

```bash
docker-compose exec openclaw-gateway bash -c 'openclaw devices list'
# ...
#Pending (1)
#┌──────────────────────────────────────┬──────────────────────────────────┬──────────┬────────────────────────────────────────────────────┬────────────┬────────┬────────┐
#│ Request                              │ Device                           │ Role     │ Scopes                                             │ IP         │ Age    │ Flags  │
#├──────────────────────────────────────┼──────────────────────────────────┼──────────┼────────────────────────────────────────────────────┼────────────┼────────┼────────┤
#│ 9c4b3ec7-73d1-4f02-9097-636e44e8c611 │ d45c8d9425868ef6561aa005daa28f50 │ operator │ operator.admin, operator.read, operator.write,     │            │ 1m ago │        │
#│                                      │ ce889e9b591b8300bcebcad37cb68f01 │          │ operator.approvals, operator.pairing               │            │        │        │
#└──────────────────────────────────────┴──────────────────────────────────┴──────────┴────────────────────────────────────────────────────┴────────────┴────────┴────────┘
# ...

docker-compose exec openclaw-gateway bash -c 'openclaw devices approve <REQUEST-ID>'
# in my case:
docker-compose exec openclaw-gateway bash -c 'openclaw devices approve 9c4b3ec7-73d1-4f02-9097-636e44e8c611'
# ...
# Approved d45c8d9425868ef6561aa005daa28f50ce889e9b591b8300bcebcad37cb68f01 (9c4b3ec7-73d1-4f02-9097-636e44e8c611)
```

Then log in again:

<p>
<img src="screenshots/1-OpenClaw-Control.jpg" width="100%">
</p>




Full permission: _data/exec-approvals.json

```json
  "defaults": {
    "security": "full",
    "ask": "off",
  },
```

Base URL
- OpenAI => https://api.openai.com
- DeepSeek => https://api.deepseek.com