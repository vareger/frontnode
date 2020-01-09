# FrontNode

**FrontNode** is project that helps you to get events from bitcoin node.

If you need to get info about transactions related to some addresses you can do it in easy way with FrontNode. Registrate your addresses in FrontNode by doing POST http query and subscribe on kafka endpoint.

Additional functionality is check your address balance (if it is not a script), you can do it by doing simple GET http query.

Lets look at the short example, but first lets look at docker-compose.yaml file first, for saving our progress of sync and other data we should uncomment volumes in all servers.
Example 'bitcoind' service line was: '''# /host_folder:/root/.bitcoin''' should be like: - '''~/btc_node_dir:/root/.bitcoin'''

After doing this we are ready for launch FrontNode enviroment, you can do it by launch command:
'''docker-compose up'''

This command will launch bitcoin node, zookeeper, kafka, postgresql. It will take tame while bitcoin-node and balance-service will be synchronized.
Then lets set up our enviroment variables to our OS or IDEA we going to use them for launch front node.
SERVER_PORT: 8080;						
ZOOKEEPER_NAMESPACE=event-loader;
ZOOKEEPER_URL=127.0.0.1:2181;
KAFKA_BOOTSTRAP_URL=localhost:9092,localhost:9093;
GROUP_ID=event_loader;
NODE_URL=http://127.0.0.1:8332; 		#endpoint to bitcoin-node RPC
NODE_LOGIN=uper; 						#bitcoin-node RPC Login
NODE_PASSWORD=writelnreal; 				#bitcoin-node RPC Password
TOPIC_ALL=bitcoin.mainnet.transactions;
TOPIC_BLOCK=bitcoin.mainnet.blocks;
TOPIC_REJECTED=bitcoin.mainnet.rejected.transactions;
TOPIC_TRANSFER=ethereum.rinkeby.events.erc20.transfer;
START_BLOCK=2084325;                    #FrontNode will start traking events since START_BLOCK.
BLOCK_LAG=0;
DB_URL=jdbc:postgresql://localhost:5434/btc-event-loader;  #PG url you can find information in docker-compose.yaml in service: 
DB_USER=postgres;
DB_PASSWORD=Fdw3asr42Js2efcxpO;
CLIENT_ID=event_loader;
AUTH_TOKEN=K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ         #Authentication token value

Finally we can build and launch our FrontNode:
'''gradle build'''
'''java -jar ./build/libs/btc-event-loader-0.1.0.jar'''

Lets test our FrontNode.
For example we whant to retrieve any Input/Output transactions related to this addresses '17A16QmavnUfCW11DAApiJxp7ARnxN5pGC' for doing this we should add this address to watch list.
For doing this we have to submit simple Post query on our front node url. Let's look at example:

'''
POST /api/v1/addresses HTTP/1.1
Host: localhost:8080
Authorization: K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ
Content-Type: application/json
Cache-Control: no-cache

{
	"address": "17A16QmavnUfCW11DAApiJxp7ARnxN5pGC"
}
'''

Now lets subscribe to our kafka topic: 'bitcoin.mainnet.transactions' by using any kafka client you want, we are ready to receive events. For checkin this you can submit some BTC from/to address you are tracking.

Lets check anouther functionality for example we whant to to broadcast transaction throw FrontNode lets do this.
For doing this we have to submit Post query on out front note url. Let's look at example:
POST /api/v1/transactions HTTP/1.1
Host: localhost:8080
Authorization: K5JK6fdG9clAktrqDvIEgZTbjybgLGgLxPymygFdRxDdSJjSBs9uRDJrxytXnIHZ
Content-Type: application/json
Cache-Control: no-cache

{
	"data": "020000000001012640e4d43ea1c2305ab33cb4b33778f767e35c5bcf41bdd42120f7d0cfc142710100000000fdffffff01124b360000000000160014777e36a83f3941ed211edc1726a3092f86cbc2f202483045022100d35d2fa189032479d7b898ea2d44f34bcdc5e6fe874d6fc68fecb7a7291c5acd022064e6a247b8e342a58e5aeb0cc4353d5a675d997ed2346a010459df62e634668b01210298a84ce462a3abe4e2d01300f3b31a87980544f734cdee9241c7601dc52f6cf5b5891800"
}

For receiving updates about this transaction we have to subscribe on the same topic as in previous example: 'bitcoin.mainnet.transactions'.