## Linkerd Proxy Internals


`linkerd2-proxy` is the main crate where the proxy's main func is present. It is where the config is loaded and the `tokio` runtime is initiated to run on the current thread