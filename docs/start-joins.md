## Mapping Another Table

Let's add a field to our `User` which is a `GraphQLObjectType`: their `Comments` which also map to a SQL table as a one-to-many relationship. Let's define the `Comment` type and map to its SQL table `comments`.

```javascript
const Comment = new GraphQLObjectType({
  name: 'Comment',
  sqlTable: 'comments',
  uniqueKey: 'id',
  fields: () => ({
    // id and body column names are the same
    id: {
      type: GraphQLInt
    },
    body: {
      type: GraphQLString
    }
  })
})
```

We need to add a field to our `User`, and tell `joinMonster` how to grab these comments via a `JOIN`. 

## Writing the JOIN Condition

This can be done with a `sqlJoin` property with a function. It will take the parent table and child table names (actually the aliases that `joinMonster` will generate) as parameters respectively and return the join condition.

```javascript
const User = new GraphQLObjectType({
  //...
  fields: () => ({
    //...
    comments: {
      type: new GraphQLList(Comment),
      // a function to generate the join condition from the table aliases
      // NOTE: you must double-quote any case-sensitive column names the table aliases are already quoted
      sqlJoin: (userTable, commentTable) => `${userTable}.id = ${commentTable}.author_id`
    }
  })
})
```

**Note:** If your column names have capital letters, or consist of anything that isn't an alpha-numeric character, $ and #, then you must *double-quote* your column names in the returned `JOIN` condition. The table aliases being passed to the `sqlJoin` function are already quoted, but any identifier that you type yourself will not be automatically quoted.

Now you can query for the comments for each user!
```graphql
{
  users { 
    id, idEncoded, email, fullName
    comments { id, body }
  }
}
```
