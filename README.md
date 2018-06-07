***There was a big change on Mar 14. If you are looking for the old one, please use v0.1.3.***

MTProto
===
Telegram MTProto and proxy (over gRPC) in Go (golang).
Telegram API layer: ***71***

## Quick start
```sh
# Run simple shell with your Telegram API id, hash, and, server address with your phone number.
# If you don't have Telegram API stuffs, get them from 'https://my.telegram.org/apps'.
go run examples/simpleshell/main.go <APIID> <APIHASH> <PHONE> <IP> <PORT>

# Then you can see 'Enter code:' message
# Telegram sends you an authentication code. Check it on your mobile or desktop app and put it.
Enter code: <YOUR_CODE>

# Now signed-in. Let's get your recent dialogs. 
# You can see them in JSON.
$ dialogs
....

# Quit the shell.
$ exit

# You can find 'credentials.json' file which keeps your MTProto secerets.
ls -al credentials.json

# You can check if the scerets correct by sign-in with it.
go run main.go credentials.json
```

## Usage
You can find the real code at [simpleshell](https://github.com/cjongseok/mtproto/blob/master/examples/simpleshell/main.go).
### Sign-in with key
```go
// Mew MTProto manager
config, _ := mtproto.NewConfiguration(appVersion, deviceModel, systemVersion, language, 0, 0, key)
manager, _ := mtproto.NewManager(config)

// Sign-in by key
mconn, _ := manager.LoadAuthentication()
```
### Sign-in without key
```go
// New MTProto manager
config, _ := mtproto.NewConfiguration(appVersion, deviceModel, systemVersion, language, 0, 0, "new-credentials.json")
manager, _ := mtproto.NewManager(config)

// Request to send an authentication code
// It needs your phone number and Telegram API stuffs you can check in https://my.telegram.org/apps 
mconn, sentCode, err := manager.NewAuthentication(phoneNumber, apiID, apiHash, ip, port)

// Get the code from user input
fmt.Scanf("%s", &code)

// Sign-in and generate the new key
_, err = mconn.SignIn(phoneNumber, code, sentCode.GetValue().PhoneCodeHash)
```
### Telegram RPC
All the methods in [TL-schema](https://github.com/cjongseok/mtproto/blob/master/compiler/scheme-71.tl) are implemented in Go.
The function list is at the *'functions'* section in [TL-schema](https://github.com/cjongseok/mtproto/blob/master/compiler/scheme-71.tl).<br>
Let's have two examples, 'messages.getDialogs' and 'messages.sendMessage'.
#### Get dialogs
```go
// New RPC caller
caller := mtproto.RPCaller{mconn}

// New input peer
// In Telegram DSL, Predicates inherit a Type.
// Here we create a Predicate, InputPeerEmpty, and wrap it with its parent Type, InputPeer.
// Please refer to Types and Predicates section for more details on types in TL-schema.
emptyPeer := &mtproto.TypeInputPeer{&mtproto.TypeInputPeer_InputPeerEmpty{&mtproto.PredInputPeerEmpty{}}

// Query to Telegram
dialogs, _ := caller.MessagesGetDialogs(context.Background(), &mtproto.ReqMessagesGetDialogs{
    OffsetDate: 0,
    OffsetId: 	0,
    OffsetPeer: emptyPeer,
    Limit: 		1,
})
```
#### Send a message to a channel
```go
// New RPC caller
caller := mtproto.RPCaller{mconn}

// New input peer
// Create a Predicate, InputPeerChannel, wraped by its parent Type, InputPeer.
channelPeer := &mtproto.TypeInputPeer{&mtproto.TypeInputPeer_InputPeerChannel{
    &mtproto.PredInputPeerChannel{
        yourChannelId, yourChannelHash,
    }}}

// Send a request to Telegram
caller.MessagesSendMessage(context.Background(), &mtproto.ReqMessagesSendMessage{
    Peer:      peer,
    Message:   "Hello MTProto",
    RandomId:  rand.Int63(),
})
```

## Proxy
You can use the proxy in two purposes:
* MTProto session sharing: Many proxy clients can use the same MTProto session on the proxy server.
* MTProto in other languages: The proxy enables various languages on its client side, since it uses gRPC.

You can find the real code at [proxy_test.go](https://github.com/cjongseok/mtproto/blob/master/proxy/proxy_test.go)
### Server
#### As a daemon
You can run the proxy as a stand-alone daemon or a container.
See [mtprotod](https://github.com/cjongseok/mtproto/tree/master/mtprotod).
#### Inside Go app
Use mtproto/proxy package.
```go
// New proxy server
config, _ := mtproto.NewConfiguration(apiId, apiHash, appVersion, deviceModel, systemVersion, language, 0, 0, key)
server = proxy.NewServer(port)

// Start the server
server.Start(config, phone)
```
### Client in Go
```go
// New proxy client
client, _ := proxy.NewClient(proxyAddr)

// Telegram RPC over proxy. It is same with the previous 'Get dialogs' but the RPC caller
emptyPeer := &mtproto.TypeInputPeer{&mtproto.TypeInputPeer_InputPeerEmpty{&mtproto.PredInputPeerEmpty{}}
dialogs, err := client.MessagesGetDialogs(context.Background(), &mtproto.ReqMessagesGetDialogs{
    OffsetDate: 0,
    OffsetId:   0,
    OffsetPeer: emptyPeer,
    Limit:      1,
})
```
### Client in Python
See [py](https://github.com/cjongseok/mtproto/tree/master/py).
### Client in other languages
By compiling [types.tl.proto](https://github.com/cjongseok/mtproto/tree/master/types.tl.proto) and [proxy/tl_update.proto](https://github.com/cjongseok/mtproto/tree/master/proxy/tl_update.proto), 
you can create clients in your preferred language.<br>
For this, you need to install [Google Protobuf](https://developers.google.com/protocol-buffers/) together with gRPC library of the target language.
Then you can compile protobuf files with these command lines:
* [Go](https://github.com/cjongseok/mtproto/blob/master/compiler/build.sh)
* [Python](https://github.com/cjongseok/mtproto/blob/master/compiler/build_py.sh)

For other languages, refer to [gRPC tutorial](https://grpc.io/docs/).

<!--
### Types and Predicates
### X and !X

## Tools
### Keygen
### Dumplayer

## Compiler
-->


## Acknowledgement
* https://github.com/sdidyk/mtproto: It is the backend of most MTProto Go implementations.
I referred its MTProto schema compiler, (de)serializer, handshaking, and encryption.
* https://github.com/shelomentsevd/mtproto: I referred its layer 65 implementation and API wrappers.
* https://github.com/ronaksoft/mtproto: I referred its backend changes for layer 71.


## License
Apache 2.0
