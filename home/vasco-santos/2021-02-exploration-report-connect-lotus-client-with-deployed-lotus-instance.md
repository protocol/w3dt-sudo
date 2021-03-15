## Connect a Lotus js client to a Lotus running instance

### Background

The [lotus JS client](https://github.com/filecoin-shipyard/js-lotus-client) is a collection of small JS libraries, which enables developers to interact with Lotus via it’s JSON-RPC API. It is decoupled in:

- [Schema](https://github.com/filecoin-shipyard/js-lotus-client-schema): contains schemas for JSON data representation compatible with Lotus JSON-RPC API
- Provider libraries ([node](https://github.com/filecoin-shipyard/js-lotus-client-provider-nodejs) and [browser](https://github.com/filecoin-shipyard/js-lotus-client-provider-browser)): inspired by “providers” in the Ethereum ecossystem, a provider links to a running node exposing interface that connects to a Lotus JSON-RPC API endpoint using WebSockets or HTTP.
- [rpc](https://github.com/filecoin-shipyard/js-lotus-client-rpc): used together with a schema for creating the RPC messages from function parameters

### Prepare lotus

The API Address is specified in the lotus configuration file (by default in `~/.lotus/config.toml`). According to the [docs](https://docs.filecoin.io/build/lotus/#getting-started-with-lotus-apis), a lotus daemon does not support remote API access by default and should be configured. After experimenting with it, and hitting the same problems it became clear that this configuration is not needed.
In addition, the configuration [doc for the remote access](https://docs.filecoin.io/build/lotus/enable-remote-api-access/#setting-the-listening-interface-for-the-api-endpoint) states that this is only relevant for miners.

For having access to the remote API, we need to guarantee that the machine allows access to the default configured port for the API listen address. If that is not the case, the port should be updated to an open one, or the machine settings should be changed (which sometimes might not be possible).

### Lotus api info

The lotus daemon has a CLI command `lotus auth`, which aims to manage the RPC permissions for clients. With this command you can create tokens `lotus auth create-token` or check the API information for connecting to a node `lotus auth api-info`.

Focus on the API information, when we run the command it fails with `--perm` flag not set, while there is no information about such flag in the CLI. This flag accepts strings like `read`, `write` and `admin`.

Running `lotus auth api-info --perm read` returns `FULLNODE_API_INFO={token}:{multiaddr}`. This misleads the user thinking that a token will be needed. However, a token is not needed if the permissions are set to `read`.

### Conclusion

As an action item, the relevant inconsistencies found while connecting a Lotus js client to a Lotus running instance will be reported to be fixed.
