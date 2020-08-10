## EventTracker
A neo-cli plugin to convert smart contract notifications into MongoDB inserts/deletes including RPC commands to query the database.
Heavily based on NotifyMongo plugin written by hal0x2328.

### Author
Merl111

### Introduction
This plugin allows a smart contract author to synchronize an external database to their smart contract state, using only smart contract notification messages.
This allows information to be more readily available to a dApp without having to query the smart contract over RPC,
and empowers the dApp's API backend with capabilities such as pagination, sorting and filtering using familiar noSQL expressions.
In addition to that the events can be queried through rpc calls.

### Features
* Select database, collection, actions for any smart contract all through JSON configuration file
* Multiple actions possible per notification
* Every inserted record includes the corresponding blockchain transaction ID automatically
* Maintains a record in a separate collection containing the most recent block height processed by the plugin

### Installation

Four possible parameters can be passed in:
1. `mode [1,2,3]`
    - `1` - 3. param is used as a to address
    - `2` - 3. param is used as a from address
    - `3` - 3. param is used as a to or from address

2. `contract hash` - the contract to query (needs to be configured with the plugin)
3. `address` - to/from address for transfer events
4. `block height` - (optional) when given only events greater than are retreived

```
curl -X POST -H 'Content-Type: application/json' -d '{"jsonrpc":"2.0","id":"id","method":"getcontractlog","params":[2,"ed07cffad18f1308db51920d99a2af60ac66a7b3","AHWZwGRvPsTdRQYL3KQzT5YWzZaWPXGdm3", 2318509]}' http://135.181.45.206:10332
```
### RPC Query
```
git clone https://github.com/neo-plugins-coz/NotifyMongo
cd NotifyMongo
dotnet publish -c Release
cp -r ./NotifyMongo {neo-cli folder}/Plugins
cp ./bin/Release/netstandard2.0/publish/NotifyMongo.dll {neo-cli folder}/Plugins
cp ./bin/Release/netstandard2.0/publish/Mongo*.dll {neo-cli folder}
```

### Configuration
Define one or more contract scripthashes and actions to perform for specific notifications coming from that contract

* `MongoHost` - host or IP address of your MongoDB server
* `MongoPort` - TCP port of your MongoDB server
* `MongoUser` - authorized MongoDB user
* `MongoPass` - authorized MongoDB user's password
* `Contracts` - list of contracts to watch for notification messages
* `Scripthash` - scripthash of the contract - will be used as the MongoDB database name
* `Actions` - list of actions to perform for matching notifications
* `OnNotification` - name of the notification message to match on
* `Action` - create/delete a MongoDB document in the collection
* `Collection` - name of the MongoDB collection to act on
* `Keyindex` - define a key field in the notification array to use in subsequent operations (index starts at 1, meaning the first field of the notification after the name. Specifying `-1` means the notification does not contain a key field)
* `Schema` - list defining how the notification array is structured, in order
* `Name` - notification array field name
* `Type` - type of data encoded in the notification field, will be converted to this format before insertion into MongoDB

#### Example config.json settings
```
{
  "PluginConfiguration": {
    "MongoHost": "127.0.0.1",
    "MongoPort": "27017",
    "MongoUser": "",
    "MongoPass": "",
    "Contracts": [
      {
        "Scripthash": "9aff1e08aea2048a26a3d2ddbb3df495b932b1e7",
        "Actions": [
          {
            "OnNotification": "transfer",
            "Action": "create",
            "Collection": "transfers",
            "Keyindex": -1,
            "Schema": [
              {"Name": "from_address", "Type": "address"},
              {"Name": "to_address", "Type": "address"},
              {"Name": "amount", "Type": "integer"}
            ]
          }
        ]
      },
      {
        "Scripthash": "642dc1c0e7c3edbfa5d3b6964afe4aca428cfc9c",
        "Actions": [
          {
            "OnNotification": "auction_created",
            "Action": "create",
            "Collection": "auctions",
            "Keyindex": 1,
            "Schema": [
              {"Name": "token_id", "Type": "integer"},
              {"Name": "start_price", "Type": "integer"},
              {"Name": "end_price", "Type": "integer"},
              {"Name": "start_block", "Type": "integer"},
              {"Name": "end_block", "Type": "integer"}
            ]
          },
          {
            "OnNotification": "auction_canceled",
            "Action": "delete",
            "Collection": "auctions",
            "Keyindex": 1,
            "Schema": [
              {"Name": "token_id", "Type": "integer"}
            ]
          },
          {
            "OnNotification": "auction_completed",
            "Action": "create",
            "Collection": "completed",
            "Keyindex": 1,
            "Schema": [
              {"Name": "token_id", "Type": "integer"},
              {"Name": "final_bid", "Type": "integer"}
            ]
          },
          {
            "OnNotification": "auction_completed",
            "Action": "delete",
            "Collection": "auctions",
            "Keyindex": 1,
            "Schema": [
              {"Name": "token_id", "Type": "integer"}
            ]
          }
        ]
      }
    ]
  }
}
```
