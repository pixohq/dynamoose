A document represents an item for a given model in DynamoDB. This item can created locally (meaning it's not yet saved in DynamoDB), or created from an item already stored in DynamoDB (ex. `Model.get`).

A document/item is similar to a row in a relational database or a document in MongoDB.

## new Model(object)

In order to create a new document you just pass in your object into an instance of your model.

```js
const User = dynamoose.model("User", {"id": Number, "name": String});
const myUser = new User({
	"id": 1,
	"name": "Tim"
});
console.log(myUser.id); // 1

// myUser is now a document instance of the User model
```

## document.save([settings,] [callback])

This saves a document to DynamoDB. This method uses the `putItem` DynamoDB API call to store your object in the given table associated with the model. This method is overwriting, and will overwrite the data you currently have in place for the existing key for your table.

This method returns a promise that will resolve when the operation is complete, this promise will reject upon failure. You can also pass in a function into the `callback` parameter to have it be used in a callback format as opposed to a promise format. Nothing will be passed into the result for the promise or callback.

You can also pass a settings object in as the first parameter. The following options are available for settings are:

| Name | Type | Default | Notes |
|---|---|---|---|
| overwrite | boolean | true | If an existing document with the same hash key should be overwritten in the database. You can set this to false to not overwrite an existing document with the same hash key. |
| return | string | `document` | If the function should return the `document` or `request`. If you set this to `request` the request that would be made to DynamoDB will be returned, but no requests will be made to DynamoDB. |
| condition | [`dynamoose.Condition`](Condition) | `null` | This is an optional instance of a Condition for the save. |

Both `settings` and `callback` parameters are optional. You can pass in a `callback` without `settings`, just by passing in one argument and having that argument be the `callback`. You are not required to pass in `settings` if you just want to pass in a `callback`.

```js
//...
const myUser = new User({
	"id": 1,
	"name": "Tim"
});

try {
	await myUser.save();
	console.log("Save operation was successful.");
} catch (error) {
	console.error(error);
}

// OR

myUser.save((error) => {
	if (error) {
		console.error(error);
	} else {
		console.log("Save operation was successful.");
	}
});
```

## document.delete([callback])

This deletes the given document from DynamoDB. This method uses the `deleteItem` DynamoDB API call to delete your object in the given table associated with the model.

This method returns a promise that will resolve when the operation is complete, this promise will reject upon failure. You can also pass in a function into the `callback` parameter to have it be used in a callback format as opposed to a promise format. Nothing will be passed into the result for the promise or callback.

```js
//...
const myUser = User.get("1");

try {
	await myUser.delete();
	console.log("Delete operation was successful.");
} catch (error) {
	console.error(error);
}

// OR

myUser.delete((error) => {
	if (error) {
		console.error(error);
	} else {
		console.log("Delete operation was successful.");
	}
});
```

## document.populate([settings], [callback])

This allows you to populate a document with document instances for the subdocuments you are referencing in your schema. This function will return a promise, or call the `callback` parameter function upon completion.

The `settings` parameter is an object you can pass in with the following properties:

| Name | Type | Description | Default |
| --- | --- | --- | --- |
| properties | string \| [string] \| boolean | Which properties should it populate. Passing `true` is equivalent to `**` & `false` is equivalent to not populating at all. | `true` \| `**` |

```js
const user = await User.get(2);

try {
	const populatedUser = await user.populate();
	console.log(populatedUser);
} catch (error) {
	console.error(error);
}

// OR

user.populate((populatedUser, error) => {
	if (error) {
		console.error(error);
	} else {
		console.log(populatedUser);
	}
});
```

## document.serialize([serializer])

This function serializes the document with the given serializer. The serializer parameter can either be a string or object. If it is an object you can pass in the same serializer as you do into [`Model.serializer.add`](Model#modelserializeraddname-serializer). If you pass in a string it will use the registered serializer with that name that is attached to the Model.

This function will return an object.

```js
const myUser = new User({"id": 1, "name": "Bob"});

myUser.serialize({"include": ["id"]}); // {"id": 1}
myUser.serialize({"exclude": ["name"]}); // {"id": 1}

myUser.serialize("myRegisteredSerializer");
```

In the event no `serializer` parameter is passed in, the default serialization for the Model will be used.

```js
const myUser = new User({"id": 1, "name": "Bob"});

myUser.serialize(); // {"id": 1, "name": "Bob"}
```

## document.original()

This function returns the original item that was received from DynamoDB. This function will return a JSON object that represents the original item. In the event no item has been retrieved from DynamoDB `null` will be returned.

```js
const user = await User.get(1);
console.log(user); // {"id": 1, "name": "Bob"}
user.name = "Tim";

console.log(user); // {"id": 1, "name": "Tim"}
console.log(user.original()); // {"id": 1, "name": "Bob"}
```

## document.toJSON()

This function returns a JSON object representation of the document. This is most commonly used when comparing a document to an object you receive elsewhere without worrying about prototypes.

```js
const user = new User({"id": 1, "name": "Tim"});

console.log(user); // Document {"id": 1, "name": "Tim"}
console.log(user.toJSON()); // {"id": 1, "name": "Tim"}
```

Due to the fact that a document instance is based on an object it is rare that you will have to use this function since you can access all the properties of the document directly. For example, both of the results will yield the same output.

```js
const user = new User({"id": 1, "name": "Tim"});

console.log(user.id); // 1
console.log(user.toJSON().id); // 1
```
