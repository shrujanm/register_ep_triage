# MCP OAuth Authentication Demo

This example demonstrates OAuth 2.0 authentication with the Model Context Protocol using **separate Authorization Server (AS) and Resource Server (RS)** to comply with the new RFC 9728 specification.

---

## Running the Servers

### Step 0: Clone the repo in the VS Code
- Save the workspace.

### Step 1: Start Authorization Server

```bash
# Navigate to the simple-auth directory
cd examples/servers/simple-auth

# Start Authorization Server on port 9000
uv run mcp-simple-auth-as --port=9000
```

**What it provides:**

- OAuth 2.0 flows (registration, authorization, token exchange)
- Simple credential-based authentication (no external provider needed)  
- Token introspection endpoint for Resource Servers (`/introspect`)

---

### Step 2: Start Resource Server (MCP Server)

```bash
# In another terminal, navigate to the simple-auth directory
cd examples/servers/simple-auth

# With RFC 8707 strict resource validation (recommended for production)
uv run mcp-simple-auth-rs --port=8001 --auth-server=http://localhost:9000  --transport=streamable-http --oauth-strict

```

### Step 3: Test with VS Code

- Create an `mcp.json` within `.vscode` directory
- Add below mentioned MCP server config within `mcp.json`

```
{
    "servers": {
        "test-mcp-for-Oauth": {
            "type": "http",
            "url": "http://localhost:8001/mcp"
        }
    }
}
```

- Start the mcp server.
- It should prompt for completing the Oauth flow as mentioned below. Click on "Allow"

  - <img width="276" height="224" alt="image" src="https://github.com/user-attachments/assets/009ca01f-6506-4bdb-b8a8-59dca1dfc818" />

- This should open up a login page, add any password and click on "Sign in"

  - <img width="583" height="385" alt="image" src="https://github.com/user-attachments/assets/55c23a1d-0605-47bc-871b-8c2ea4c60782" />

- The Oauth flow should be completed and below message should be visible on browser.

  - <img width="1492" height="479" alt="image" src="https://github.com/user-attachments/assets/08ae6a57-781f-4a2e-97da-94cf5489be98" />

- On the `mcp.json`, we should see the server should have been started with 1 tool.

  - <img width="305" height="92" alt="image" src="https://github.com/user-attachments/assets/d95489ce-751a-4e27-ac61-78143d63f7b6" />


### Step 4: Observe the logs for Resource Server (MCP Server) during Client initiation:

```
simple-auth % uv run mcp-simple-auth-rs --port=8001 --auth-server=http://localhost:9000  --transport=streamable-http --oauth-strict
IntrospectionTokenVerifier initialized with endpoint: http://localhost:9000/introspect, server_url: http://localhost:8001/mcp, resource_url: http://localhost:8001/mcp, validate_resource: True
INFO:mcp_simple_auth.server:🚀 MCP Resource Server running on http://localhost:8001/
INFO:mcp_simple_auth.server:🔑 Using Authorization Server: http://localhost:9000/
INFO:     Started server process [14310]
INFO:     Waiting for application startup.
INFO:mcp.server.streamable_http_manager:StreamableHTTP session manager started
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:8001 (Press CTRL+C to quit)

INFO:     127.0.0.1:61276 - "POST /mcp HTTP/1.1" 401 Unauthorized
INFO:     127.0.0.1:61277 - "GET /.well-known/oauth-protected-resource/mcp HTTP/1.1" 200 OK

INFO:mcp_simple_auth.token_verifier:Sending introspection request to http://localhost:9000/introspect
INFO:httpx:HTTP Request: POST http://localhost:9000/introspect "HTTP/1.1 200 OK"
INFO:mcp_simple_auth.token_verifier:Received introspection response: status=200
INFO:mcp_simple_auth.token_verifier:Validating resource: requested=http://localhost:8001/mcp, configured=http://localhost:8001/mcp
INFO:mcp.server.streamable_http_manager:Created new transport with session ID: 2b7e170ce0114b09af664053f64d494e
INFO:     127.0.0.1:61285 - "POST /mcp HTTP/1.1" 200 OK

INFO:mcp_simple_auth.token_verifier:Sending introspection request to http://localhost:9000/introspect
INFO:mcp_simple_auth.token_verifier:Sending introspection request to http://localhost:9000/introspect
INFO:mcp_simple_auth.token_verifier:Sending introspection request to http://localhost:9000/introspect
INFO:mcp_simple_auth.token_verifier:Sending introspection request to http://localhost:9000/introspect
INFO:httpx:HTTP Request: POST http://localhost:9000/introspect "HTTP/1.1 200 OK"
INFO:mcp_simple_auth.token_verifier:Received introspection response: status=200
INFO:mcp_simple_auth.token_verifier:Validating resource: requested=http://localhost:8001/mcp, configured=http://localhost:8001/mcp
INFO:httpx:HTTP Request: POST http://localhost:9000/introspect "HTTP/1.1 200 OK"
INFO:httpx:HTTP Request: POST http://localhost:9000/introspect "HTTP/1.1 200 OK"
INFO:mcp_simple_auth.token_verifier:Received introspection response: status=200
INFO:mcp_simple_auth.token_verifier:Validating resource: requested=http://localhost:8001/mcp, configured=http://localhost:8001/mcp
INFO:mcp_simple_auth.token_verifier:Received introspection response: status=200
INFO:mcp_simple_auth.token_verifier:Validating resource: requested=http://localhost:8001/mcp, configured=http://localhost:8001/mcp
INFO:httpx:HTTP Request: POST http://localhost:9000/introspect "HTTP/1.1 200 OK"
INFO:     127.0.0.1:61288 - "POST /mcp HTTP/1.1" 200 OK

WARNING:root:Failed to validate request: Received request before initialization was complete
INFO:     127.0.0.1:61287 - "POST /mcp HTTP/1.1" 202 Accepted

INFO:mcp_simple_auth.token_verifier:Received introspection response: status=200
INFO:mcp_simple_auth.token_verifier:Validating resource: requested=http://localhost:8001/mcp, configured=http://localhost:8001/mcp
INFO:     127.0.0.1:61290 - "GET /mcp HTTP/1.1" 200 OK

INFO:     127.0.0.1:61289 - "POST /mcp HTTP/1.1" 200 OK

INFO:mcp.server.lowlevel.server:Processing request of type ListPromptsRequest

```

### Step 5: Observe the logs for Oauth Server during Client initiation:

- Few things to note here is - Localized VS Code always brings in its own client id.
- That might the cause for VS Code to assume that it's existing client id is pre-registered with the Oauth server and skips `/register` endpoint
- We were able to mimic the same behavious with MCP Inspector as well where if the client id was provided within the configuration, it skipped calling the `/register` endpoint. If the client id was not provided, it will call the `/register` endpoint of Oauth server and get itself a new client_id from the Oauth server

```
simple-auth % uv run mcp-simple-auth-as --port=9000
INFO:mcp_simple_auth.auth_server:🚀 MCP Authorization Server running on http://localhost:9000/
INFO:     Started server process [14166]
INFO:     Waiting for application startup.
INFO:     Application startup complete.
INFO:     Uvicorn running on http://localhost:9000 (Press CTRL+C to quit)

INFO:     127.0.0.1:61278 - "GET /.well-known/oauth-authorization-server HTTP/1.1" 200 OK

WARNING:mcp_simple_auth.simple_auth_provider:Client ID f33e2bba-ffe4-4b8f-8a41-0e0897e0ecd9 not found. Temporarily creating a public client.
INFO:mcp_simple_auth.simple_auth_provider:Registered temporary public client: redirect_uris=[AnyUrl('http://127.0.0.1:33418/'), AnyUrl('http://localhost:33418/'), AnyUrl('http://127.0.0.1:33418/callback/'), AnyUrl('http://localhost:33418/callback/'), AnyUrl('https://vscode.dev/redirect'), AnyUrl('http://localhost:6274/oauth/callback/debug'), AnyUrl('http://localhost:6274/oauth/callback')] token_endpoint_auth_method='client_secret_post' grant_types=['authorization_code'] response_types=['code'] scope='user' client_name='Client f33e2bba-ffe4-4b8f-8a41-0e0897e0ecd9' client_uri=None logo_uri=None contacts=None tos_uri=None policy_uri=None jwks_uri=None jwks=None software_id=None software_version=None client_id='f33e2bba-ffe4-4b8f-8a41-0e0897e0ecd9' client_secret=None client_id_issued_at=None client_secret_expires_at=None

Authorize called with client: redirect_uris=[AnyUrl('http://127.0.0.1:33418/'), AnyUrl('http://localhost:33418/'), AnyUrl('http://127.0.0.1:33418/callback/'), AnyUrl('http://localhost:33418/callback/'), AnyUrl('https://vscode.dev/redirect'), AnyUrl('http://localhost:6274/oauth/callback/debug'), AnyUrl('http://localhost:6274/oauth/callback')] token_endpoint_auth_method='client_secret_post' grant_types=['authorization_code'] response_types=['code'] scope='user' client_name='Client f33e2bba-ffe4-4b8f-8a41-0e0897e0ecd9' client_uri=None logo_uri=None contacts=None tos_uri=None policy_uri=None jwks_uri=None jwks=None software_id=None software_version=None client_id='f33e2bba-ffe4-4b8f-8a41-0e0897e0ecd9' client_secret=None client_id_issued_at=None client_secret_expires_at=None, params: state='7OWewmH/wzwGzDUiS1mGyw==' scopes=['user'] code_challenge='KnnQxb_uApcx7_1tTJG-Zol5wcNDuOd2pA5CuSDqutc' redirect_uri=AnyUrl('http://127.0.0.1:33418/') redirect_uri_provided_explicitly=True resource='http://localhost:8001/mcp', state: 7OWewmH/wzwGzDUiS1mGyw==

INFO:     ::1:61280 - "GET /authorize?client_id=f33e2bba-ffe4-4b8f-8a41-0e0897e0ecd9&response_type=code&code_challenge=KnnQxb_uApcx7_1tTJG-Zol5wcNDuOd2pA5CuSDqutc&code_challenge_method=S256&scope=user&resource=http%3A%2F%2Flocalhost%3A8001%2Fmcp&redirect_uri=http%3A%2F%2F127.0.0.1%3A33418%2F&state=7OWewmH%2FwzwGzDUiS1mGyw%3D%3D HTTP/1.1" 302 Found

INFO:     ::1:61281 - "GET /login?state=7OWewmH/wzwGzDUiS1mGyw==&client_id=f33e2bba-ffe4-4b8f-8a41-0e0897e0ecd9 HTTP/1.1" 200 OK

Handling callback for user: demo_user, state: 7OWewmH/wzwGzDUiS1mGyw==
Available states: dict_keys(['7OWewmH/wzwGzDUiS1mGyw=='])
INFO:     ::1:61280 - "POST /login/callback HTTP/1.1" 302 Found

INFO:     127.0.0.1:61284 - "POST /token HTTP/1.1" 200 OK
INFO:     ::1:61286 - "POST /introspect HTTP/1.1" 200 OK
INFO:     ::1:61291 - "POST /introspect HTTP/1.1" 200 OK
INFO:     ::1:61292 - "POST /introspect HTTP/1.1" 200 OK
INFO:     ::1:61293 - "POST /introspect HTTP/1.1" 200 OK
INFO:     ::1:61294 - "POST /introspect HTTP/1.1" 200 OK
```

### Step 6: Mimicing this behaviour with MCP Inspector:

- Initiating Oauth with pre-configured Client ID - as you can see, it did not initiate any `/register` call to the Oauth server. It used the same client id to call the `/authorize` endpoint.

  <img width="1502" height="793" alt="image" src="https://github.com/user-attachments/assets/cdea5252-accf-4b28-a720-4cb9d1b48c26" />

- Here is an example - where client id was not readily provided to the MCP Inspector. This time, it went ahead and called `/register` endpoint, performed DCR and got itself a Client ID post which it was able to move forward with `/authorize` call.

  <img width="1509" height="791" alt="image" src="https://github.com/user-attachments/assets/8566fa45-33ea-4654-9ad1-bd6d8d49ce62" />

- The question now is - Can we change the behaviour of VS Code client to not consider its existing Client ID it has for connecting with any Custom MCP Servers and Custom Oauth server.  

