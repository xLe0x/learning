---
banner: "![[tom-barrett-7FNOH-qSxMI-unsplash.jpg]]"
banner_icon:
banner_icon: 🕸️
banner_y: 0.276
---

<iframe title="Learn How To Hack GraphQL (Zero 2 Hero)" src="https://www.youtube.com/embed/QcAPUWXbncc?feature=oembed" height="113" width="200" allowfullscreen="" allow="fullscreen" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;"></iframe>
<iframe title="Hacking GraphQL | Easy $$$$ By Breaking Logic Workflow" src="https://www.youtube.com/embed/00rZJFMrLlE?feature=oembed" height="113" width="200" allowfullscreen="" allow="fullscreen" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;"></iframe>
## Finding GraphQL endpoints
Before you can test a GraphQL API, you first need to find its endpoint. As GraphQL APIs use the same endpoint for all requests, this is a valuable piece of information.

## Universal queries

**If you send `query{__typename}` to any GraphQL endpoint, it will include the string `{"data": {"__typename": "query"}}` somewhere in its response**. This is known as a universal query, and is a useful tool in probing whether a URL corresponds to a GraphQL service.

The query works because every GraphQL endpoint has a reserved field called `__typename` that returns the queried object's type as a string.

## Common endpoint names

- `/graphql`
- `/api`
- `/api/graphql`
- `/graphql/api`
- `/graphql/graphql`

**if these endpoints didn't return any GraphQL response try to append `/v1`.**

==*GraphQL services will often respond to any non-GraphQL request with a <mark style="background: #D2B3FDA6;">"query not present"</mark> or similar error. You should bear this in mind when testing for GraphQL endpoints.*==

## Request methods

It is best practice for production GraphQL endpoints to only accept <mark style="background: #FFB8EBA6;">POST</mark> requests that have a content-type of `application/json`, as this helps to protect against CSRF vulnerabilities. However, some endpoints may accept alternative methods, such as GET requests or POST requests that use a content-type of `x-www-form-urlencoded`.

==If you can't find the GraphQL endpoint by sending POST requests to common endpoints, try resending the universal query using alternative HTTP methods.==


## Exploiting unsanitized arguments

If the API uses arguments to access objects directly, it may be vulnerable to access control vulnerabilities. A user could potentially access information they should not have simply by supplying an argument that corresponds to that information. This is sometimes known as an insecure direct object reference (IDOR).

For example, the query below requests a product list for an online shop:

```php
#Example product query 
query { 
products { 
		id
		name
		listed 
	}
}
```

The product list returned contains only listed products.

```php
#Example product response 
{ 
"data": { 
	"products": [ 
			{ "id": 1, "name": "Product 1", "listed": true }, 
			{ "id": 2, "name": "Product 2", "listed": true }, 
			{ "id": 4, "name": "Product 4", "listed": true } 
		] 
	} 
}
```

From this information, we can infer the following:

- Products are assigned a sequential ID.
- Product ID 3 is missing from the list, possibly because it has been delisted.

By querying the ID of the missing product, we can get its details, even though it is not listed on the shop and was not returned by the original product query.


```php
#Query to get missing product query 
{ 
	product(id: 3) { 
		id 
		name 
		listed 
	} 
}
```

```php
#Missing product response 
{ 
"data": { 
	"product": {
		"id": 3, "name": "Product 3", "listed": no }
	}
}
```

