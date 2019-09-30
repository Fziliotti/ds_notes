Sistemas Distribuídos na Prática
======

Este é um conjunto de notas de um curso de Sistemas Distribuídos introdutório, usando uma abordagem prática orientada a um projeto de sistemas de banco de dados NOSQL como motivação. A versão publicada destas notas está disponível [aqui](https://lasarojc.github.io/ds_notes/).

A estrutura intendida no final deste primeiro semestre em que as notas estão sendo estruturadas em MD é a seguinte.
Os links abaixo podem estar quebrados. Para ver a versão atual, use o índice à esquerda.


# Introdução

* [Motivação](./intro/motivacao.md)
* [O quê e por quê?](./intro/definicao.md)
* [Como distribuir?](./intro/middleware.md)
* [Tipos e Arquiteturas](./intro/tipos.md)

# Comunicação

* [Sockets](./basics/socket.md)
* [Invocação Remota de Procedimentos](./basics/rpc.md)
* [Estudo de Caso: gRPC](./basics/grpc.md)
* [Estudo de Caso: Thrift](./TODO.md)
* [Estudo de Caso: RMI](./TODO.md)
* [Projeto](./projeto/client_server.md)
* [*Publish/Subscribe*](./TODO.md)

# Concorrência

* [*Multiprocessing* e *Multithreading*](./basics/multiprogramming.md)
* [Concorrência](./concorrencia/concorrencia.md)

# Peer-2-Peer

* [Tabelas de Espalhamento Distribuídas](./p2p/dht.md)
* [Estruturas de Dados para P2P e outros SD](./p2p/ed_sd.md)
* [DynamoDB Deep Dive](./p2p/dynamo.md)
* [Projeto](./projeto/p2p.md)

# Micro-serviços

# Coordenação

* [Coordenação](./coordenacao/coordenacao.md)
* [Tempo Físico](./tempo/fisico.md)
* [Tempo Lógico](./tempo/logico.md)



# Tolerância a Falhas
* [Dependabilidade](./ft/dependabilidade.md)
* [Modelos](./ft/modelos.md)
* [Comunicação em Grupo](./ft/comunicao_grupo.md)
* [Replicação de Máquinas de Estados](./ft/smr.md)
* [Estudo de Caso: Raft](./fr/raft.md)
* [Estudo de Caso: Zookeeper](./ft/zookeeper.md)
* [Estudo de Caso: Atomix](./ft/atomix.md)
* [Projeto](./projeto/replicated.md)

# Tecnologias "Quentes"

* [Arquitetura Reativa](./reactive.md)
* [Map & Reduce](./mapreduce.md)
