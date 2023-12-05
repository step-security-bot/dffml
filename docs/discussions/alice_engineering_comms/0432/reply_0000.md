## 2023-10-26 @pdxjohnny Engineering Logs

- https://spacetimedb.com/docs/server%20module%20languages/rust/index
- https://spacetimedb.com/docs/client%20sdk%20languages/python/sdk%20reference

```bash
echo '--jwt-pub-key-path The path to the public jwt key for verifying identities (SPACETIMEDB_JWT_PUB_KEY)'
echo '--jwt-priv-key-path  The path to the private jwt key for issuing identities (SPACETIMEDB_JWT_PRIV_KEY)'
echo 'OIDC interop ^ ?'
spacetime start --in-memory --listen-addr 127.0.0.1:7777 2>&1 1>/dev/null &
spacetime server add "http://127.0.0.1:7777" local-in-memory-0
spacetime server set-default local-in-memory-0
```

```console
$ spacetime init --lang rust .
```

**src/lib.rs**

```rust
// In this small example, we have two rust imports:
// |spacetimedb::spacetimedb| is the most important attribute we'll be using.
// |spacetimedb::println| is like regular old |println|, but outputting to the module's logs.
use spacetimedb::{spacetimedb, ReducerContext, println};

// This macro lets us interact with a SpacetimeDB table of Person rows.
// We can insert and delete into, and query, this table by the collection
// of functions generated by the macro.
#[spacetimedb(table)]
pub struct Person {
    name: String,
}

#[spacetimedb(init)]
pub fn init() {
    // Called when the module is initially published
}

#[spacetimedb(connect)]
pub fn identity_connected(_ctx: ReducerContext) {
    // Called everytime a new client connects
}

#[spacetimedb(disconnect)]
pub fn identity_disconnected(_ctx: ReducerContext) {
    // Called everytime a client disconnects
}

// This is the other key macro we will be using. A reducer is a
// stored procedure that lives in the database, and which can
// be invoked remotely.
#[spacetimedb(reducer)]
pub fn add(name: String) {
    // |Person| is a totally ordinary Rust struct. We can construct
    // one from the given name as we typically would.
    let person = Person { name };

    // Here's our first generated function! Given a |Person| object,
    // we can insert it into the table:
    Person::insert(person);
}

// Here's another reducer. Notice that this one doesn't take any arguments, while
// |add| did take one. Reducers can take any number of arguments, as long as
// SpacetimeDB knows about all their types. Reducers also have to be top level
// functions, not methods.
#[spacetimedb(reducer)]
pub fn say_hello() {
    // Here's the next of our generated functions: |iter()|. This
    // iterates over all the columns in the |Person| table in SpacetimeDB.
    for person in Person::iter() {
        // Reducers run in a very constrained and sandboxed environment,
        // and in particular, can't do most I/O from the Rust standard library.
        // We provide an alternative |spacetimedb::println| which is just like
        // the std version, excepted it is redirected out to the module's logs.
        println!("Hello, {}!", person.name);
        // log::info!("Hello, {}!", person.name);
    }
    println!("Hello, World!");
    // log::info!("Hello, World!");
}

// Reducers can't return values, but can return errors. To do so,
// the reducer must have a return type of `Result<(), T>`, for any `T` that
// implements `Debug`.  Such errors returned from reducers will be formatted and
// printed out to logs.
#[spacetimedb(reducer)]
pub fn add_person(name: String) -> Result<(), String> {
    if name.is_empty() {
        return Err("Name cannot be empty".to_string());
    }

    Person::insert(Person { name });

    Ok(())
}
```

- Install wasm optimizer

```bash
export REPO_ORG="WebAssembly" \
  && export REPO_NAME="binaryen" \
  && export LATEST_VERSION=$(gh api "https://api.github.com/repos/${REPO_ORG}/${REPO_NAME}/releases/latest" | jq -r '.tag_name') \
  && curl -sfL "https://github.com/${REPO_ORG}/${REPO_NAME}/releases/download/${LATEST_VERSION}/binaryen-${LATEST_VERSION}-x86_64-linux.tar.gz" \
  | tar -C "${HOME}/.local/bin/" --strip-components=2 -xvz ${REPO_NAME}-${LATEST_VERSION}/bin/
```

```bash
cargo build
spacetime publish --project-path . people
for person in $(echo alice bob); do spacetime call people add_person "${person}"; done
```

```console
$ spacetime publish --project-path . people
info: component 'rust-std' for target 'wasm32-unknown-unknown' is up to date
checking crate with spacetimedb's clippy configuration
    Checking spacetime-module v0.1.0 (/home/pdxjohnny/Documents/rust/spacetime-test)
    Finished release [optimized] target(s) in 0.38s
    Finished release [optimized] target(s) in 0.18s
Created new database with domain: person, address: 5ebd1221afb10aaf61d6c8a527413d82
$ spacetime logs people
 INFO: spacetimedb: Creating table `Person`
 INFO: spacetimedb: Invoking `init` reducer
 INFO: spacetimedb: Database initialized
$ for person in $(echo alice bob); do spacetime call people add_person "${person}"; done
$ spacetime call people say_hello
$ spacetime logs people
 INFO: spacetimedb: Creating table `Person`
 INFO: spacetimedb: Invoking `init` reducer
 INFO: spacetimedb: Database initialized
 INFO: src/lib.rs:56: Hello, bob!
 INFO: src/lib.rs:56: Hello, alice!
 INFO: src/lib.rs:59: Hello, World!
$ spacetime sql people "SELECT * FROM Person"
 name
-------
 bob
 alice
```

- https://github.com/clockworklabs/SpacetimeMultiplayerDemo/blob/0884427dd2e19c8354271f2e1f2a68d74f4247f0/Client/Assets/_Project/Game/DemoGameManager.cs#L40-L98
  - GitHub Actions Runner is C#
  - repository_dispatch
    - Automation
  - workflow_dispatch
    - manual
- This looks like an interesting option for `BovinePubSub`
  - https://spacetimedb.com/docs/client%20sdk%20languages/python/sdk%20reference#async-client-reference

> Function | Description
> -- | --
> Function SpacetimeDBAsyncClient::run | Run the client. This function will not return until the client is closed.
> Function SpacetimeDBAsyncClient::subscribe | Subscribe to receive data and transaction updates for the provided queries.
> Function SpacetimeDBAsyncClient::register_on_subscription_applied | Register a callback when the local cache is updated as a result of a change to the subscription queries.
> Function SpacetimeDBAsyncClient::force_close | Signal the client to stop processing events and close the connection to the server.
> Function SpacetimeDBAsyncClient::schedule_event | Schedule an event to be fired after a delay

---

[![asciicast](https://asciinema.org/a/617367.svg)](https://asciinema.org/a/617367)

```
tests/test_federation_activitypub_bovine.py
Service private key written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/bob/workspace/storage/service_private_key.pem                                [46/52]
Service parameters written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/bob/workspace/service_parameters.json                                                                                            
Service parameters: /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/bob/workspace/service_parameters.json                                                                                                      
 * Serving Quart app 'scitt_emulator.server'                                                                     
 * Debug mode: True                                                                                                                                                                                                                
 * Please use an ASGI server (e.g. Hypercorn) directly in production                                             
 * Running on http://127.0.0.1:0 (CTRL + C to quit)                                                              
Adding new user to config.toml                          
Please add did:key:z6MkhwCVH2EcJmqiy5meDXKNJoUrNaeLd1F4UcfVn2kqZosk to the access list of your ActivityPub actor                                                                                                                   
[2023-10-26 22:11:58 -0700] [12260] [INFO] Running on http://127.0.0.1:44103 (CTRL + C to quit)                                                                                                                                    
Service private key written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/alice/workspace/storage/service_private_key.pem                                                                                 
Service parameters written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/alice/workspace/service_parameters.json                                                                                          
Service parameters: /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/alice/workspace/service_parameters.json                                                                                                    
 * Serving Quart app 'scitt_emulator.server'                                                                     
 * Debug mode: True                                     
 * Please use an ASGI server (e.g. Hypercorn) directly in production                                             
 * Running on http://127.0.0.1:0 (CTRL + C to quit)                                                                                                                                                                                
Adding new user to config.toml                                                                                                                                                                                                     
Please add did:key:z6MkmtJZ66g6hF3cbZCAAJbHu9oscRBxi5s3nApuPwEQvZ3Q to the access list of your ActivityPub actor                                                                                                                   [2023-10-26 22:11:58 -0700] [12274] [INFO] Running on http://127.0.0.1:40113 (CTRL + C to quit)                                                                                                                                    
A COSE signed Claim was written to:  /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/claim.cose                                                                                                                
[2023-10-26 22:11:58 -0700] [12260] [INFO] 127.0.0.1:49268 POST /entries 1.1 503 115 2882                                                                                                                                          
Operation b6e63d2a-5092-4ef0-ad81-2c6f565b7469 created                                                                                                                                                                             
A COSE signed Claim was written to:  /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/bob/workspace/storage/operations/b6e63d2a-5092-4ef0-ad81-2c6f565b7469.cose                                                [2023-10-26 22:11:59 -0700] [12260] [INFO] 127.0.0.1:49268 POST /entries 1.1 202 83 8799                                                                                                                                           
Leaf hash: 228a20bfda08ca68dbe4d95d132b9d617e60b871faa361a62e351837226a6398                                                                                                                                                        
Root: e61a8d3c6ce8a37b737faab9df26bf68d45e4c06524cb7d09e22fb7e2d2b740c                                           
Receipt written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/bob/workspace/storage/sha384:11685a0fd1cb413db8f982773a48326c87a58620bcc0d074a9406d47a1a8de5c3503e5bea6db494ed1776853499675b8.receipt.cbor  
A COSE signed Claim was written to:  /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/bob/workspace/storage/sha384:11685a0fd1cb413db8f982773a48326c87a58620bcc0d074a9406d47a1a8de5c3503e5bea6db494ed1776853499675b8.cose                                                
[2023-10-26 22:12:00 -0700] [12260] [INFO] 127.0.0.1:49268 GET /operations/b6e63d2a-5092-4ef0-ad81-2c6f565b7469 1.1 200 205 6406                                                                                                   
[2023-10-26 22:12:00 -0700] [12260] [INFO] 127.0.0.1:49268 GET /entries/sha384:11685a0fd1cb413db8f982773a48326c87a58620bcc0d074a9406d47a1a8de5c3503e5bea6db494ed1776853499675b8/receipt 1.1 200 770 4000
Claim Registered:                                                                                                                                                                                                                  
  json:     {'operationId': 'b6e63d2a-5092-4ef0-ad81-2c6f565b7469', 'status': 'running'}                                                                                                                                             Entry ID: sha384:11685a0fd1cb413db8f982773a48326c87a58620bcc0d074a9406d47a1a8de5c3503e5bea6db494ed1776853499675b8                                                                                                                
  Receipt:  .//tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/claim.receipt.cbor                                                                                                                               
Entry ID written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/claim.entry_id.txt                                                                                                                         
A COSE signed Claim was written to:  /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/claim.cose                                                                                                                
Operation 5c3b676a-1fe9-4553-b023-2c2e1b9a07af created                                                                                                                                                                             
A COSE signed Claim was written to:  /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/alice/workspace/storage/operations/5c3b676a-1fe9-4553-b023-2c2e1b9a07af.cose
[2023-10-26 22:12:01 -0700] [12274] [INFO] 127.0.0.1:60922 POST /entries 1.1 202 83 4649                                                                                                                                           
[2023-10-26 22:12:02 -0700] [12274] [INFO] 127.0.0.1:60922 GET /operations/5c3b676a-1fe9-4553-b023-2c2e1b9a07af 1.1 503 115 4380                                                                                                   
Leaf hash: 094dec521d864b2b61489c1f60aae642564b5c5b45df2ea0f1468f94bca997db                                                                                                                                                        
Root: da6c73c998d43ccef5f3508daecd675a2b0b2ead0125457943392f5cdc63f43f                                           
Receipt written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/alice/workspace/storage/sha384:ee813058619ea51b078cc4633dcde892d92eff3538f4556d15bac3d0db9f3459cbd4b4d3c3a933d25ebcfcda1c4e84e3.receipt.cbor
A COSE signed Claim was written to:  /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/alice/workspace/storage/sha384:ee813058619ea51b078cc4633dcde892d92eff3538f4556d15bac3d0db9f3459cbd4b4d3c3a933d25ebcfcda1c4
e84e3.cose                                                                                                                                                                                                                         
[2023-10-26 22:12:03 -0700] [12274] [INFO] 127.0.0.1:60922 GET /operations/5c3b676a-1fe9-4553-b023-2c2e1b9a07af 1.1 200 205 11735                                                                                                  
[2023-10-26 22:12:03 -0700] [12274] [INFO] 127.0.0.1:60922 GET /entries/sha384:ee813058619ea51b078cc4633dcde892d92eff3538f4556d15bac3d0db9f3459cbd4b4d3c3a933d25ebcfcda1c4e84e3/receipt 1.1 200 770 5697
Claim Registered:                                       
  json:     {'operationId': '5c3b676a-1fe9-4553-b023-2c2e1b9a07af', 'status': 'running'}                                                                                                                                           
  Entry ID: sha384:ee813058619ea51b078cc4633dcde892d92eff3538f4556d15bac3d0db9f3459cbd4b4d3c3a933d25ebcfcda1c4e84e3                                                                                                                
  Receipt:  .//tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/claim.receipt.cbor                                                                                                                               
Entry ID written to /tmp/pytest-of-pdxjohnny/pytest-140/test_docs_federation_activityp0/claim.entry_id.txt                                                                                                                         
[2023-10-26 22:12:03 -0700] [12260] [INFO] 127.0.0.1:33354 GET /entries/sha384:ee813058619ea51b078cc4633dcde892d92eff3538f4556d15bac3d0db9f3459cbd4b4d3c3a933d25ebcfcda1c4e84e3 1.1 404 193 2060
HTTP error 404: {                                       
  "detail": "Entry sha384:ee813058619ea51b078cc4633dcde892d92eff3538f4556d15bac3d0db9f3459cbd4b4d3c3a933d25ebcfcda1c4e84e3 not found",                                                                                             
  "type": "urn:ietf:params:scitt:error:entryNotFound"                                                            
}                                                       

[2023-10-26 22:12:03 -0700] [12274] [INFO] 127.0.0.1:46174 GET /entries/sha384:11685a0fd1cb413db8f982773a48326c87a58620bcc0d074a9406d47a1a8de5c3503e5bea6db494ed1776853499675b8 1.1 404 193 2056
HTTP error 404: {                                       
  "detail": "Entry sha384:11685a0fd1cb413db8f982773a48326c87a58620bcc0d074a9406d47a1a8de5c3503e5bea6db494ed1776853499675b8 not found",                                                                                             
  "type": "urn:ietf:params:scitt:error:entryNotFound"                                                            
}                                                       
```

- Currently failing (expected), now need to go fix why the receipt is not getting federated from Bob to Alice