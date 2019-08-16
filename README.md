# aws-lambda-limit

AWS Lambda has a [6.00 MiB limit](https://docs.aws.amazon.com/lambda/latest/dg/limits.html) on size of the reponse payload for synchronous invocation. But what exactly does that mean?

## Example Error

You might see an error message like this if your Lambda function returns more than 6 MB of data through a callback or Promise.

```json
{
  "errorMessage": "body size is too long"
}
```

```json
{
  "errorMessage": "Exceeded maximum allowed payload size (6291556 bytes).",
  "errorType": "RequestEntityTooLarge"
}
```

## Return Types

The exact definition of the limit varies based on what is being returned.

### `string` or `Buffer`

It's easy to determine the size of JavaScript `string` and `buffer` types -- just check the `.length` property. For these return types, the limit is 6291456 bytes (6.29 MB or 6.0 [MiB](https://en.wikipedia.org/wiki/Mebibyte)).  

### `object`

An additional 100 bytes are granted if your Lambda function returns an object, which brings the limit up to 6291556 bytes.

The payload size is approximately the size of the minified JSON string (no whitespace) representing the object.
This can be computed with `JSON.stringify(object).length`. However, that only works as expected with numbers and boolean values. Strings in the response object incur some additional overhead -- perhaps about 2 bytes per string.

_Note: If viewing the response object in the AWS web-based Lambda Console, the object is shown in prettified JSON. This does not represent the actual size though._ 

### `boolean[]`

The largest allowable array of `boolean[]` contains 1,258,311 `true` values corresponding to 6291556 bytes of minified JSON,
or 1,048,592 `false` values corresponding to 6291553 bytes of minified JSON.

### `number[]`

The largest allowable array of `number[]` contains 3,145,776 single-digit numbers. As the size of numbers increases, the number of allowable elements is reduced.

### `string[]`

For an example string size of 16 characters, the largest allowable array of `string[]` contains 334,134 elements representing 6291547 bytes.
Each character that needs to be escaped within a string requires an additional byte for the ` \ ` escape character.
