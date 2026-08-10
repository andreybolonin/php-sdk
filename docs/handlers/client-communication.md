# Talking back to the client

MCP supports various ways a server can communicate back to a client on top of the main
request-response flow.

## ClientGateway

Every communication back to client is handled using the `Mcp\Server\ClientGateway` and its dedicated methods per
operation. Reach it through method argument injection for `RequestContext`. (A `ClientGateway`-typed parameter is
injected too, but unlike `RequestContext` it is not excluded from the generated input schema, so it would show up as
an argument of your tool.)

Every reference of a MCP element, that translates to an actual method call, can just add an type-hinted argument for the
`RequestContext` and the SDK will take care to include the gateway in the arguments of the method call:

```php
use Mcp\Capability\Attribute\McpTool;
use Mcp\Server\RequestContext;

class MyService
{
    #[McpTool(name: 'my_tool', description: 'My Tool Description')]
    public function myTool(RequestContext $context): string
    {
        $context->getClientGateway()->log(...);
```

## Sampling

With [sampling](https://modelcontextprotocol.io/specification/2025-06-18/client/sampling) servers can request clients to
execute "completions" or "generations" with a language model for them:

```php
$result = $clientGateway->sample('Roses are red, violets are', 350, 90, ['temperature' => 0.5]);
```

The `sample` method accepts four arguments:

1. `message`, which is **required** and accepts a string, an instance of `Content` or an array of `Mcp\Schema\Content\SamplingMessage` instances.
2. `maxTokens`, which defaults to `1000`
3. `timeout` in seconds, which defaults to `120`
4. `options` which might include `systemPrompt`, `preferences` for model choice, `includeContext`, `temperature`, `stopSequences` and `metadata`

[Find more details to sampling payload in the specification.](https://modelcontextprotocol.io/specification/2025-06-18/client/sampling#protocol-messages)

## Logging

The [Logging](https://modelcontextprotocol.io/specification/2025-06-18/server/utilities/logging) utility enables servers
to send structured log messages as notification to clients:

```php
use Mcp\Schema\Enum\LoggingLevel;

$clientGateway->log(LoggingLevel::Warning, 'The end is near.');
```

## Progress

With a [Progress](https://modelcontextprotocol.io/specification/2025-06-18/basic/utilities/progress#progress)
notification a server can update a client while an operation is ongoing:

```php
$clientGateway->progress(4.2, 10, 'Downloading needed images.');
```

## Notification

Lastly, the server can push all kind of notifications, that extend the abstract `Mcp\Schema\JsonRpc\Notification` class
to the client to:

```php
$clientGateway->notify($yourNotification);
```
