# FrontNode

**FrontNode** is project that helps you to get events from bitcoin node.

If you need to get info about transactions related to some addresses you can do it in easy way with FrontNode. Registrate your addresses in FrontNode by running POST http query and subscribe for kafka endpoint.

Additional functionality is ability to check your address balance (if it is not a script), you can do it by running simple GET http query.


##### There is project architecture diagram given below.

![Architecture diagram](./img/2.png)

# How to install

To start lets look at **docker-compose.yaml** file first.

To save your progress of sync and other data you should set volumes in all servers.

### Example:
**"bitcoind"** service line has: 

```
# volumes:
# /host_folder:/root/.bitcoin
```
you should modify it to: 

```
volumes:
~/btc_node_dir:/root/.bitcoin
```
![](./img/1.png)

After volumes are seted you are ready to launch FrontNode enviroment, by running the following command:

```
docker-compose up
```

This command will launch bitcoin node, zookeeper, kafka and postgresql. It will take some time while bitcoin-node and balance-service will be synchronized.
Then you need to set up enviroment variables into the prefered OS or IDEA you are going to use to launch front node.

```java
SERVER_PORT: 8080;						
ZOOKEEPER_NAMESPACE=event-loader;
ZOOKEEPER_URL=127.0.0.1:2181;
KAFKA_BOOTSTRAP_URL=localhost:9092,localhost:9093;
GROUP_ID=event_loader;
NODE_URL=http://127.0.0.1:8332; 		//endpoint to bitcoin-node RPC
NODE_LOGIN=uper; 						//bitcoin-node RPC Login
NODE_PASSWORD=writelnreal; 				//bitcoin-node RPC Password
TOPIC_ALL=bitcoin.mainnet.transactions;
TOPIC_BLOCK=bitcoin.mainnet.blocks;
TOPIC_REJECTED=bitcoin.mainnet.rejected.transactions;
TOPIC_TRANSFER=ethereum.rinkeby.events.erc20.transfer;
START_BLOCK=2084325;                    //FrontNode will start traking events since START_BLOCK.
BLOCK_LAG=0;
DB_URL=jdbc:postgresql://localhost:5434/btc-event-loader;  //PG url you can find information in docker-compose.yaml in service: 
DB_USER=postgres;
DB_PASSWORD=Fdw3asr42Js2efcxpO;
CLIENT_ID=event_loader;
AUTH_TOKEN=K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ //Authentication token value
```


Finally you can build and launch our FrontNode:

```java
gradle build
java -jar ./build/libs/btc-event-loader-0.1.0.jar
```




# How To Use



*  To retrieve any Input/Output transactions related to the specific addresses you should add this address to watch list by submiting simple Post query on our front node url. 


```java
POST /api/v1/addresses HTTP/1.1
Host: localhost:8080
Authorization: K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ
Content-Type: application/json
Cache-Control: no-cache

{
	"address": "17A16QmavnUfCW11DAApiJxp7ARnxN5pGC"
}
```

* To subscribe to our kafka topic: `bitcoin.mainnet.transactions` use any kafka client you want. Than you are ready to receive events. You may submit some BTC from/to address you are tracking to test it.

* To broadcast transaction throw FrontNode you should submit Post query on out front note url. 


```java
POST /api/v1/transactions HTTP/1.1
Host: localhost:8080
Authorization: K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ
Content-Type: application/json
Cache-Control: no-cache

{
	"data": "020000000001012640e4d43ea1c2305ab33cb4b33778f767e35c5bcf41bdd42120f7d0cfc142710100000000fdffffff01124b360000000000160014777e36a83f3941ed211edc1726a3092f86cbc2f202483045022100d35d2fa189032479d7b898ea2d44f34bcdc5e6fe874d6fc68fecb7a7291c5acd022064e6a247b8e342a58e5aeb0cc4353d5a675d997ed2346a010459df62e634668b01210298a84ce462a3abe4e2d01300f3b31a87980544f734cdee9241c7601dc52f6cf5b5891800"
}
```

* To receive updates about this transaction you should subscribe on the same topic as in previous example: `bitcoin.mainnet.transactions`.

* To add on traking multiple addresses by one request you should submit the following http query:

```java
POST /api/v1/addresses/batch HTTP/1.1
Host: localhost:8080
Authorization: K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ
Content-Type: application/json
Cache-Control: no-cache
[
  {
    "address": "string"
  }
]
```

* To delete address from tracking list you should submit the following http query: 

```java
DELETE /api/v1/addresses/{address} HTTP/1.1
Host: localhost:8080
Authorization: K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ
Content-Type: multipart/form-data
Cache-Control: no-cache
```

* To delete multiple addresses you should submit the following http query:

```java
DELETE /api/v1/addresses HTTP/1.1
Host: localhost:8080
Authorization: K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ
Content-Type: multipart/form-data
Cache-Control: no-cache

[
  {
    "address": "string"
  }
]
```

# Support
Have a question? Contact us frontnode@vareger.com