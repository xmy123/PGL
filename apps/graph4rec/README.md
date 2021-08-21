# Graph4Rec: An universal toolkit with Graph Neural Networks for Recommender System

**Graph4Rec** is an universal industrial toolkit with Graph Neural Networks for Recommender System.

## Requirements

 - paddlepaddle-gpu==2.1.2

 - pgl==2.1.4

## Performance

TBD

## Usage

### Input Format

TBD: data format

### PGL Graph Engine Launching

Now we support distributed loading graph data using **PGL Graph Engine**. We also develop a simple tutorial to show how to launch a graph engine, please refer to [here](../../tutorials/working_with_distributed_graph_engine.ipynb).

To launch a distributed graph service, please follow the steps below.

#### IP address setting

The first step is to set the IP list for each graph server. Each IP address with port represents a server. In `ip_list.txt` file, we set up 4 graph server as follow for demo:

```
127.0.0.1:8553
127.0.0.1:8554
127.0.0.1:8555
127.0.0.1:8556
```

#### Launching Graph Engine by OpenMPI

Before launching the graph engine, you should set up the below hyper-parameters in configuration file in `./user_configs/`:

```
etype2files: "u2buy2t:./Rec15/u2buy2i.txt,u2click2i:./Rec15/u2click2i.txt"
ntype2files: "u:./Rec15/node_types.txt,i:./Rec15/node_types.txt"
symmetry: True
shard_num: 1000
```

Then, we can launch the graph engine with the help of OpenMPI.

```
mpirun -np 4 python -m pgl.distributed.launch --ip_config ./ip_list.txt --conf ./config.yaml --mode mpi --shard_num 100
```

#### Launching Graph Engine manually

If you didn't install OpenMPI, you can launch the graph engine manually. 

Fox example, if we want to use 4 servers, we should run the following command separately on 4 terminals.

```
# terminal 3
python -m pgl.distributed.launch --ip_config /your/path/to/ip_list.txt --conf ./user_configs/multi_metapath2vec.yaml --shard_num 100 --server_id 3

# terminal 2
python -m pgl.distributed.launch --ip_config /your/path/to/ip_list.txt --conf ./user_configs/multi_metapath2vec.yaml --shard_num 100 --server_id 2

# terminal 1
python -m pgl.distributed.launch --ip_config /your/path/to/ip_list.txt --conf ./user_configs/multi_metapath2vec.yaml --shard_num 100 --server_id 1

# terminal 0
python -m pgl.distributed.launch --ip_config /your/path/to/ip_list.txt --conf ./user_configs/multi_metapath2vec.yaml --shard_num 100 --server_id 0
```

Note that the `shard_num` should be the same as in configuration file.

After successfully launching the graph engine, you can run the below command to train the model in different mode.

### Single GPU Training and Inference

```
# Training
cd ./src
export CUDA_VISIBLE_DEVICES=0
python train.py --config ../user_configs/multi_metapath2vec.yaml --ip /your/path/to/ip_list.txt

# Inference
python infer.py --config ../user_configs/multi_metapath2vec.yaml \
                --ip /your/path/to/ip_list.txt \
                --save_dir /your/path/to/save_embed \
                --infer_from /your/path/of/trained_model
```

### Distributed CPU Training and Inference

```
# Training
cd ./src
CPU_NUM=12 fleetrun --log_dir /your/path/to/fleet_logs \
                    --worker_num 4 \
                    --server_num 4 \
                    dist_cpu_train.py --config ../user_configs/multi_metapath2vec.yaml \
                                      --ip /your/path/to/ip_list.txt

# Inference
CPU_NUM=12 fleetrun --log_dir /your/path/to/fleet_logs_infer \
                    --worker_num 4 \
                    --server_num 4 \
                    dist_cpu_infer.py --config ../user_configs/multi_metapath2vec.yaml \
                                      --ip /your/path/to/ip_list.txt \
                                      --save_dir /your/path/to/save_embed \
                                      --infer_from /your/path/of/trained_model
```

Note that the `worker_num` and `server_num` in inference stage should be the same as in training stage.

The training log will be saved in `/your/path/to/fleet_logs`.