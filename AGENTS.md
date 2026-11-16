# For agents: boot GLM-5.3 K4 on four DGX Sparks

You are booting this recipe for someone. Follow these steps in order, and run each check. If you cannot run a check
yourself, ask the person to run it and paste the output. If this file and `compose.yaml` or the `Dockerfile` disagree,
follow those files and tell the person.

## 1. Check each box

On all four boxes:

- It is a DGX Spark or another GB10 box (128 GB): `nvidia-smi` names a GB10.
- Docker has the NVIDIA runtime: `docker info | grep -i nvidia`.
- About 110 GB is free where Docker keeps volumes: `df -h "$(docker info --format '{{.DockerRootDir}}')"`.
- Hugging Face answers: `curl -sI https://huggingface.co`.
- No other model runs: `docker ps` shows no model container, and `nvidia-smi` shows no GPU process.
- The ConnectX ports are up: `ls /dev/infiniband` lists devices, and `ip -br addr` shows an address on each port the
  boxes are joined by.
- Proactive compaction is off: `sysctl vm.compaction_proactiveness` prints 0 (README.md, What you need).

## 2. Configure

Copy `.env.example` to `.env` on every box. If the boxes run mentat (`docker ps` shows a `mentat` container), leave it
as it is. Otherwise fill in `PEERS` (the four LAN addresses, head first) and, if the boxes have more than one ConnectX
address each, `IPS` and `IPS2`. Use the same `.env` on every box.

## 3. Start and check

1. `docker compose up -d` on all four boxes.
2. `docker compose logs -f` on the head. The first boot fetches the model (about 100 GB a box), then the boxes
   handshake and load. It is ready when the log says the cluster serves.
3. On the head: `docker compose exec gr8 /opt/gr8/gr8 glm53 smoke`. Every check must pass.
4. On the head: `curl http://localhost:8100/v1/models` lists the model.

If a box's log says another engine is running, run `docker compose exec gr8 /opt/gr8/gr8 glm53 down` on the head,
wait until `pgrep -f 'bin/g6m4'` finds nothing on every box, and start again.
