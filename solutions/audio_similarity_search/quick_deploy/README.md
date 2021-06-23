# Reverse Image Search Based on Milvus and ResNet50

## Overview

This demo uses PANNs, which is a pretrained audio neural network model, and Milvus to build a system that can perform audio searches.

PANNs source: https://github.com/qiuqiangkong/audioset_tagging_cnn

## Data source

This demo uses the TUT Acoustic scenes 2017 Evaluation dataset, which contains 1622 10-second audio clips that fall within 15 categories: Bus, Cafe, Car, City center, Forest path, Grocery store, Home, Lakeside beach, Library, Metro station, Office, Residential area, Train, Tram, and Urban park. Download and extract the audio files in an accessible location.

Dataset size: ~ 3.7 GB.

Download location: https://zenodo.org/record/1040168#.YNJ1YC9h1KM

> Note: You can also use other audio files for testing.

## How to deploy the system

### 1. Start Milvus and MySQL

The system will use Milvus to store and search the feature vector data, and Mysql is used to store the correspondence between the ids returned by Milvus and the audio file paths.

- **Start Milvus v1.1.0**

Copy and paste the commands below, or refer to the Install Milvus v1.1.0 documentation for more information on how to run dockerized Milvus.

```bash
$ wget -P /home/$USER/milvus/conf https://raw.githubusercontent.com/milvus-io/milvus/v1.1.0/core/conf/demo/server_config.yaml
$ sudo docker run -d --name milvus_cpu_1.1.0 \
-p 19530:19530 \
-p 19121:19121 \
-v /home/$USER/milvus/db:/var/lib/milvus/db \
-v /home/$USER/milvus/conf:/var/lib/milvus/conf \
-v /home/$USER/milvus/logs:/var/lib/milvus/logs \
-v /home/$USER/milvus/wal:/var/lib/milvus/wal \
milvusdb/milvus:1.1.0-cpu-d050721-5e559c
```

> Note the version of Milvus.

- **Start MySQL**

```bash
$ docker run -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.7
```

### 2. Start Server

The next step is to start the system server, which will provide HTTP backend services.

- **Install the Python packages**

```bash
$ cd server
$ pip install -r requirements.txt
```

- **Set configuration**

```bash
$ vim server/src/config.py
```

Please modify the parameters according to your own environment. Here listing some parameters that need to be set, for more information please refer to [config.py](./server/src/config.py).

| **Parameter**    | **Description**                                       | **Default setting** |
| ---------------- | ----------------------------------------------------- | ------------------- |
| MILVUS_HOST      | The IP address of Milvus, you can get it by ifconfig. | 127.0.0.1           |
| MILVUS_PORT      | Port of Milvus.                                       | 19530               |
| VECTOR_DIMENSION | Dimension of the vectors.                             | 2048                |
| MYSQL_HOST       | The IP address of Mysql.                              | 127.0.0.1           |
| MYSQL_PORT       | Port of Milvus.                                       | 3306                |
| DEFAULT_TABLE    | The milvus and mysql default collection name.         | milvus_audio_search   |

- **Run the code**

Then start the server with Fastapi.

```bash
$ cd src
$ python main.py
```

- **The API docs**

Type 127.0.0.1:5000/docs in your browser to see all the APIs.

[img] (all_API.png)

- **Code  structure**

If you are interested in our code or would like to contribute code, feel free to learn more about our code structure.

```
└───server
│   │   Dockerfile
│   │   requirements.txt
│   │   main.py  # File for starting the program.
│   │
│   └───src
│       │   config.py  # Configuration file.
│       │   encode.py  # Covert audio files to embeddings.
│       │   milvus_helpers.py  # Connect to Milvus server and insert/drop/query vectors in Milvus.
│       │   mysql_helpers.py   # Connect to MySQL server, and add/delete/query IDs and object information.
│       │   
│       └───operations # Call methods in milvus.py and mysql.py to insert/query/delete objects.
│               │   insert.py
│               │   query.py
│               │   delete.py
│               │   count.py
```

- **How to use**

The server will accept HTTP requests at the API endpoints listed on the documentation, which can be found at http://server_ip_address:port/docs. If you set up the server locally with the default settings, the address will be http://127.0.0.1:5000/docs.

In general, audio files must first be loaded with /audio/load before they can be searched with /audio/search.

You can also use the docs page to test API requests through a user interface.

Loading audio files:

[img] (insert.png)

Searching audio files:

[img] (search.png)

If you have any questions about Milvus, would like assistance setting up the project, or are interested in learning more about new updates, please join us on our slack channel at http://milvusio.slack.com/
