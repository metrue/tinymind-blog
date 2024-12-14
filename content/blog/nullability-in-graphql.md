---
title: Nullability in GraphQL Schema Design
date: 2022-11-03T05:35:07.322Z
---

### Background

**Nullability** in GraphQL is different from how we handle null values in other API solutions. In REST or gRPC APIs, we expect an endpoint to either return an object, a list, or other piece(s) of data as a whole, or not at all. If a REST GET operation fails, we expect the entire request to have failed, not just a subset of the operation. 
However, in a GraphQL type system, every field is nullable by default. It’s entirely acceptable and likely that most of your data graph will load successfully, but a small part could fail due to the composition of the many micro-services responsible for populating all of that data. 
### Nullability and Query Result
_TL;DR, **if your query operation includes a non-nullable field, when it is null or its resolver throws an exception, the entire query fails**._

Take a look at a simple example, we have micro services in the backend and the schema defines 3 fields for the Query type as 3 entry points into the GraphQL API. 
- **getUserName** which is non-nullable, its resolver throws an exception directly.
- **getUserGender** which is nullable, its resolver also throws an exception directly.
- **helloworld** which is a normal field, its resolve return a simple string “🚀🚀🚀🚀🚀🚀🚀 GraphQL 🚀🚀🚀🚀🚀🚀🚀”

```javascript
type Query {
  getUserName: String!
  getUserGender: String
  helloworld: String
}
 
const resolvers = {
 Query: {
   getUserName: () : string => {
     throw new Error('resolve exception')
   },
   getUserGender: () : string => {
     throw new Error('resolve exception')
   },
   helloworld: () => {
     return '🚀🚀🚀🚀🚀🚀🚀 GraphQL 🚀🚀🚀🚀🚀🚀🚀';
   }
 }
};
```
- **“null” on non-nullable field in a query**
Following operation is to inquire **getUserName** and **helloworld** in one query.
```javascritp
query HelloWorld {
   getUserName
   helloworld
}
```
We gets the result for the query operation,
```javascript
{
   "errors": [
       {
           "message": "resolve exception", 
           …..
           "extensions": {
               "code": "INTERNAL_SERVER_ERROR",
               "exception": {
                …… 
               }
           }
       }
   ],
   "data": null
}
```
- **“null” on nullable in a query**
Following operation is to inquire **getUserGender** and **helloworld** in one query.
```javascript
query HelloWorld {
   getUserGender
   helloworld
}
```
We gets the result for the query operation,
```javascript
{
   "errors": [
       {
           "message": "resolve exception", 
           …..
           "extensions": {
               "code": "INTERNAL_SERVER_ERROR",
               "exception": {
                …… 
               }
           }
       }
   ],
   "data": {
       "getUserGender": null,
       "helloworld": "🚀🚀🚀🚀🚀🚀🚀 GraphQL 🚀🚀🚀🚀🚀🚀🚀"
   }
}

```
We can see the “errors” field of the response is the same, it includes the detailed information of the resolving exception that happened in the GraphQL API server. But the data fields of the response are different,
In the first query operation, since an exception happened in resolving the non-nullable field “getUserName”, the whole data field became “**null**” directly, **the whole query failed**.
In the second case, the “getUserGender” is nullable, although an exception happened in resolving, in the data field of response, only the “getUserGener” part is  “null”, the other parts of data are healthy, **only part of the query failed**.

### Conclusion
Nullability in GraphQL unlocks improvements in resiliency, graceful UI degradation, frictionless user workflows, and other benefits. When a field or fields in a type are marked as “non-nullable” (using the !). a failure can no longer occur gracefully.
**For scheme design**, one aspect of planning your data types is nullability, and this is important to get it  right. Using nullable fields in your GraphQL projections means that when (not if) one or more parts of the response fail, the rest of the response may remain perfectly usable. It means only part of the screen fails, the rest of the screen may remain perfectly usable for users. Sometimes, for the same data query, providing nullable and non-nullable of a root type field for clients to fetch based on their business case might be an option.
**For query operation design**, we should be aware of the risk that the whole query operation may fail due to a non-nullable field getting “null” (database goes down, network glitch, service exception, etc), depending on the situation, you can discuss with data provider to update the schema or split the big query operation into small ones if needed.

### References
- [GraphQL Best Practices](https://graphql.org/learn/best-practices/#nullability)
