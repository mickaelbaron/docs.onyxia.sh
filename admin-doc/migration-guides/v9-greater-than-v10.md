# ⬆️ v9 -> v10

There is no breaking for the configuration. You should be able to migrate easily if you use Onyxia in a standard way.

However, there is breaking change in the API of Onyxia, so if you use some custom client that directly connect to onyxia-api which is not recommanded you may notice some removed endpoints.



{% hint style="info" %}
**New functionnality**\
You can use overwrite schemas based on role. You can configure role inside Onyxia and provide schema for each role. For more details, refer to the documentation: [Onyxia v10 Catalog](../catalog-of-services/).
{% endhint %}
