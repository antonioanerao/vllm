# mpac-vllm-repo

Este repositório contém uma stack de inferência com **vLLM** para servir o modelo **Qwen/Qwen3-4B-FP8** em Docker com aceleração NVIDIA, exposição via Traefik e configuração por variáveis de ambiente.

## Arquivos de exemplo

- `Dockerfile.qwen3-4b-fp8.example`
- `docker-compose.qwen3-4b-fp8.example.yml`
- `.env.example`

## Visão geral da stack

### Container

O arquivo `Dockerfile.qwen3-4b-fp8.example` usa como base a imagem oficial:

- `vllm/vllm-openai:latest`

Além disso:

- define `HF_HOME=/models`
- define `VLLM_WORKER_MULTIPROC_METHOD=spawn`
- cria o diretório `/models`
- atualiza o pacote `transformers`
- expõe a porta `8000`

O modelo não é baixado no build. O download ocorre em tempo de execução, usando o token do Hugging Face e o cache montado em volume.

### Compose

O arquivo `docker-compose.qwen3-4b-fp8.example.yml`:

- sobe o serviço `vllm`
- usa `entrypoint: ["vllm", "serve"]`
- publica a porta interna `8000`
- monta `./models:/models`
- reserva `shm_size: "8gb"` para melhorar a comunicação entre processos
- configura `tensor parallel` com 2 GPUs
- protege a API com múltiplas `--api-key`
- expõe o serviço via Traefik usando o host definido em variável de ambiente

### Modelo servido

O exemplo atual serve:

- `Qwen/Qwen3-4B-FP8`

com os seguintes destaques:

- `--tensor-parallel-size 2`
- `--max-model-len 75000`
- `--hf-overrides` com `rope_scaling` do tipo `yarn`
- `--reasoning-parser qwen3`
- duas chaves de API aceitas pelo servidor

## Variáveis de ambiente

O projeto espera um arquivo `.env` com variáveis como:

- `HF_TOKEN`
- `VLLM_API_KEY_1`
- `VLLM_API_KEY_2`
- `NVIDIA_VISIBLE_DEVICES`
- `VLLM_HOST`

Exemplo de uso:

```env
HF_TOKEN=hf_token_aqui
VLLM_API_KEY_1=token_interno_1
VLLM_API_KEY_2=token_interno_2
NVIDIA_VISIBLE_DEVICES=GPU-uuid-1,GPU-uuid-2
VLLM_HOST=transcreveai2.mpac.mp.br
```

* Para listar o ID de todas as GPUs, use o comando `nvidia-smi -L`

## Como subir

Use os arquivos de exemplo como base para criar seus arquivos reais:

```bash
cp Dockerfile.qwen3-4b-fp8.example Dockerfile
cp docker-compose.qwen3-4b-fp8.example.yml docker-compose.yml
cp .env.example .env
```

Depois ajuste o `.env` e suba a stack:

```bash
docker compose up -d --build
```

## Acesso à API

O serviço expõe a API compatível com OpenAI do vLLM na porta `8000`.

Endpoints úteis:

- `GET /v1/models`
- `POST /v1/chat/completions`
- `POST /v1/completions`

Como o servidor usa `--api-key`, as requisições devem enviar:

```http
Authorization: Bearer SUA_CHAVE
```

Exemplo:

```bash
curl http://127.0.0.1:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SUA_CHAVE" \
  -d '{
    "model": "Qwen/Qwen3-4B-FP8",
    "messages": [
      {
        "role": "user",
        "content": "Explique resumidamente machine learning."
      }
    ],
    "temperature": 0.2,
    "max_tokens": 200
  }'
```

## Observações

- O cache de modelos fica persistido no volume `./models`.
- Para multi-GPU, prefira usar UUID em `NVIDIA_VISIBLE_DEVICES`.
- Se o token do Hugging Face for exposto, gere outro imediatamente.
