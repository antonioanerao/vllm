# mpac-vllm-repo

Este repositório contém uma stack de inferência com **vLLM** para servir o modelo **Qwen/Qwen3-4B-FP8** em container Docker com aceleração NVIDIA.

## Visão geral da stack

A configuração atual é baseada em dois arquivos de exemplo:

- `Dockerfile.qwen3-4b-fp8.example`
- `docker-compose.qwen3-4b-fp8.example.yml`

### Componentes

- **Base de container**: `nvidia/cuda:12.1.1-cudnn8-runtime-ubuntu22.04`
- **Runtime Python**: `python3` + `pip`
- **Servidor de inferência**: pacote `vllm` (instalado via `pip`)
- **Modelo**: `Qwen/Qwen3-4B-FP8` (baixado do Hugging Face)
- **Exposição HTTP**: porta `8000`
- **Orquestração**: Docker Compose com runtime NVIDIA

## Como o container é montado

No `Dockerfile.qwen3-4b-fp8.example`:

1. Usa imagem CUDA 12.1.1 com cuDNN 8 em Ubuntu 22.04.
2. Define variáveis de ambiente:
   - `HF_HOME=/models`
   - `TRANSFORMERS_CACHE=/models`
   - `VLLM_WORKER_MULTIPROC_METHOD=spawn`
3. Instala dependências de sistema: `python3`, `python3-pip`, `git`, `curl`.
4. Instala `vllm` via `pip3`.
5. Cria diretório `/models`.
6. Faz pré-download do modelo `Qwen/Qwen3-4B-FP8` com `huggingface_hub.snapshot_download`.
7. Expõe a porta `8000`.
