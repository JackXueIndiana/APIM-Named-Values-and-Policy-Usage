# APIM-Named-Values-and-Policy-Usage
APIM Named Value is hosting name/value pair and particularly the value is a string. 'https://azure.github.io/apim-lab/apim-lab/4-policy-expressions/policy-expressions-4-4-named-values.html' states that Named Values (aka Properties) are a collection of key/value pairs that are global to the service instance. These properties can be used to manage string constants across all API configurations and policies. Values can be expressions, secrets (encrypted by APIM), or Key Vault, which links to a corresponding secret in Azure Key Vault.

The usage of Named Value is documented in 'https://learn.microsoft.com/en-us/azure/api-management/api-management-howto-properties?tabs=azure-portal'.

While you can use named values in policy expressions, the other way around is not supported 'https://feedback.azure.com/d365community/idea/506de38e-cea5-ec11-a81c-6045bd7d1bee'. However, there are two workarounds may be helpful: 

## Workaround 1: Using self-chained calls:
Per 'https://techcommunity.microsoft.com/t5/azure-paas-blog/self-chained-apim-request-limitation-in-internal-virtual-network/ba-p/1940417', self-chained APIM request limitation in internal Virtual network mode (Developer and Premium tier) - Microsoft Community Hub, you can add the needed Values from the properties into Header/Body and thus the context of the next call. In other words,
- In the initial request, extract relevant data (e.g., headers or query parameters).
- Use this data to create subsequent requests within the same policy.
- Pass data from the initial request to subsequent requests using named values or context variables.
Here is an example:
~~~
<inbound>
    <!-- Initial request -->
    <base />
    <set-variable name="myValue" value="@(context.Request.Headers.GetValueOrDefault("X-Custom-Header"))" />

    <!-- Subsequent request -->
    <send-request mode="new" response-variable-name="response2">
        <set-url>https://api.example.com/endpoint2?param1={{myValue}}</set-url>
    </send-request>

    <!-- Use response2 in subsequent policies -->
    <!-- ... -->
</inbound>
~~~
In case, you need to use an array, then you can build the array with desired values and delimitator and parse them out in the next call.

## Workaround 2: Using set-variable to bring in the named value:
Based on this blog (there are some errors/missing pieces in it 'https://fgheysels.github.io/json-in-apim-policies/')
How to use named values with JSON content in Azure API Management Policies – Frederik Gheysels' WebLog – Blog on software development, C#, Azure, IoT ... (fgheysels.github.io)

If you define a Named Value pairs
 

In your policy you write:
~~~
<set-variable name="mynamedvalues" value="{{mynamedvalue}}" />
<set-header name="x-request-received-namevalues" exists-action="override">
<value>@{
        var c = (string)context.Variables["mynamedvalues"]; 
        return c;
    }</value>
</set-header>
<set-header name="x-request-received-value" exists-action="override">
    <value>{{test}}</value>
</set-header>
~~~
You can expect this output:
~~~
x-request-received-namevalues: {"product-a": "somevalue","product-b": "anothervalue","product-c": "foobar"}
x-request-received-value: 12345A
~~~

