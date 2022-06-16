---
title: Ejecutar un nodo testnet
description: Configurar y ejecutar un nodo testnet con Docker
tags:
  - tutorial
---

## Introducción

Este procedimiento demuestra cómo ejecutar un nodo local testnet usando imágenes Docker.

:::caution
Este procedimiento se centra en sistemas operativos de tipo Unix (Linux y MacOS). Este procedimiento no ha sido probado en Windows.
:::

## Prerrequisitos

Ejecutar un nodo no tiene requerimientos de hardware especiales. Los usuarios han tenido éxito ejecutando nodos en las dispostivos Raspberry Pi y otras arquitecturas de sistema on-chip. Para completar este procedimiento, debe tener instalado el siguiente software:

- [Docker](https://docs.docker.com/get-docker/)
- [curl](https://curl.se/download.html)
- [jq](https://stedolan.github.io/jq/download/)

### Configuración del firewall

Para que los servicios del nodo API funcionen correctamente, debe configurar cualquier regla de firewall de red para permitir el tráfico en los puertos discutidos en esta sección. Los detalles de la configuración de red y firewall son muy específicos para su máquina y red, por lo que no se proporciona un ejemplo detallado.

Los siguientes puertos deben abrirse en la máquina host:

Entrada:

- stacks-blockchain (abierto a `0.0.0.0/0`):
  - `20443 TCP`
  - `20444 TCP`

Salida:

- `18332`
- `18333`
- `20443-20444`

Estos puertos de salida son para sincronizar las cabeceras de [`stacks-blockchain`][] y Bitcoin. Si no están abiertos, la sincronización fallará.

## Paso 1: configuración inicial

In order to run the testnet node, you must download the Docker images and create a directory structure to hold the persistent data from the services. Download and configure the Docker images with the following commands:

```sh
docker pull blockstack/stacks-blockchain
```

Create a directory structure for the service data with the following command:

```sh
mkdir -p ./stacks-node/persistent-data/stacks-blockchain/testnet && cd stacks-node
```

## Paso 2: ejecutar Stacks blockchain

Start the [`stacks-blockchain`][] container with the following command:

```sh
docker run -d --rm \
  --name stacks-blockchain \
  -v $(pwd)/persistent-data/stacks-blockchain/testnet:/root/stacks-node/data \
  -p 20443:20443 \
  -p 20444:20444 \
  blockstack/stacks-blockchain \
/bin/stacks-node testnet
```

You can verify the running [`stacks-blockchain`][] container with the command:

```sh
docker ps --filter name=stacks-blockchain
```

## Paso 3: verificar los servicios

:::info
The initial burnchain header sync can take several minutes, until this is done the following commands will not work
:::

To verify the [`stacks-blockchain`][] burnchain header sync progress:

```sh
docker logs stacks-blockchain
```

The output should be similar to the following:

```
INFO [1626290705.886954] [src/burnchains/bitcoin/spv.rs:926] [main] Syncing Bitcoin headers: 1.2% (8000 out of 2034380)
INFO [1626290748.103291] [src/burnchains/bitcoin/spv.rs:926] [main] Syncing Bitcoin headers: 1.4% (10000 out of 2034380)
INFO [1626290776.956535] [src/burnchains/bitcoin/spv.rs:926] [main] Syncing Bitcoin headers: 1.7% (12000 out of 2034380)
```

To verify the [`stacks-blockchain`][] tip height is progressing use the following command:

```sh
curl -sL localhost:20443/v2/info | jq
```

If the instance is running you should recieve terminal output similar to the following:

```json
{
  "peer_version": 4207599105,
  "pox_consensus": "12f7fa85e5099755a00b7eaecded1aa27af61748",
  "burn_block_height": 2034380,
  "stable_pox_consensus": "5cc4e0403ff6a1a4bd17dae9600c7c13d0b10bdf",
  "stable_burn_block_height": 2034373,
  "server_version": "stacks-node 2.0.11.2.0-rc1 (develop:7b6d3ee+, release build, linux [x86_64])",
  "network_id": 2147483648,
  "parent_network_id": 118034699,
  "stacks_tip_height": 509,
  "stacks_tip": "e0ee952e9891709d196080ca638ad07e6146d4c362e6afe4bb46f42d5fe584e8",
  "stacks_tip_consensus_hash": "12f7fa85e5099755a00b7eaecded1aa27af61748",
  "genesis_chainstate_hash": "74237aa39aa50a83de11a4f53e9d3bb7d43461d1de9873f402e5453ae60bc59b",
  "unanchored_tip": "32bc86590f11504f17904ee7f5cb05bcf71a68a35f0bb3bc2d31aca726090842",
  "unanchored_seq": 0,
  "exit_at_block_height": null
}
```

## Detener el nodo testnet

Utilice los siguientes comandos para detener el nodo local testnet:

```sh
docker stop stacks-blockchain
```

## Lecturas adicionales

<!-- markdown-link-check-disable -->

- [Ejecutar un nodo API de Stacks](https://docs.hiro.so/get-started/running-api-node)
- [Ejecutando un nodo Stacks mainnet](running-mainnet-node)
<!-- markdown-link-check-enable-->