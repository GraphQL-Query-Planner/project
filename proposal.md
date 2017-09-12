# GraphQL Query Planner + App

Members:

- Derek Stride  100955939
- Ismail Syed 100923110
- Justin Krol 100941980

Supervisor:

- Babak Esfandiari


## Objectives

The objective of the project is to create a static analysis tool that can analyze a GraphQL query document and produce information about the execution [1]. Information that would be of interest includes which backend is used in the field resolution and information regarding what queries it will make.

The feature should have an API that is consistent with the current style of the gem (graphql-ruby). It should play nice with plugins that deal with optimizing the underlying queries such as batch loaders [2]. Common problems such as N+1 queries should be easily identifiable.

Additional features could include execution details like time to resolve queries of the backends used during field resolution.


## Background

### GraphQL

GraphQL is a specification [3] that defines a query language and execution engine for those queries. Applications map their business logic to a GraphQL system creating a strongly typed schema that can be queried and executed.

Since GraphQL isn’t tied to a specific backend, the implementation of GraphQL varies based on the developer ecosystem and common practices, this has lead many implementations to use a resolver based approach to executing GraphQL queries. Using resolvers for each field has lead to one GraphQL query potentially using multiple backends to fulfill the request.

### Query Optimization

A common problem for GraphQL libraries is unoptimized query execution because of the lack of assumptions that can be made about data and the associations. As an example the following GraphQL query can easily produce N+1 queries when using a relational database as the backend. These kind of problems are likely fairly easy to identify as a developer but also should be detectable by a Query Planner.

<table>
  <tr>
  <td>GraphQL</td>
  <td>

```graphql
query {
  products(first: 3) {
    image {
      src
    }
  }
}
```

  </td>
  </tr>
  <tr>
  <td>N+1 Queries</td>
  <td>

```sql
SELECT id FROM products LIMIT 3;
SELECT src FROM images WHERE product_id = 1;
SELECT src FROM images WHERE product_id = 2;
SELECT src FROM images WHERE product_id = 3;
```

  </td>
  </tr>
  <tr>
  <td>Optimized Queries</td>
  <td>

```sql
SELECT id FROM products LIMIT 3;
SELECT src FROM images WHERE product_id IN (1, 2, 3);
```

  </td>
  </tr>
</table>

## Description

### Query Planner

The first step to building out this feature is to build out test cases for how we want the feature to perform. The first case would be a simple query like the one above and the basic details about it, like what queries will be made. A test environment with mysql or a similar relational database should be set up so that we can execute the test cases.

<table>
  <tr>
  <td>

```ruby
MySchema.plan(query_string, context: ctx)
```

  </td>
  </tr>
  <tr>
  <td>

```graphql
query {
  products(first: 3) {
    image {
      src
    }
  }
}
```

  </td>
  </tr>
</table>

Later on more complicated cases should be set up, e.g. identifying which indexes are being used, queries with multiple backends, supporting different types of backends.

### Web App

<table>
  <tr>
  <td>

```ruby
OptimizedSchema.plan(query_string)
```

  </td>
  <td>

```ruby
BadSchema.plan(query_string)
```

  </td>
  </tr>
  <tr>
  <td>

```ruby
[
 {
  engine: 'mysql',
  field: 'products(first: 3)',
  query: 'SELECT id FROM products LIMIT 3;'
 },
 {
  engine: 'mysql',
  field: 'image { src }',
  query: 'SELECT src FROM images WHERE product_id IN (1, 2, 3);'
 }
]
```

  </td>
  <td>

```ruby
[
 {
  engine: 'mysql',
  field: 'products(first: 3)',
  query: 'SELECT id FROM products LIMIT 3;'
 },
 {
  engine: 'mysql',
  field: 'image { src }',
  query: 'SELECT src FROM images WHERE product_id = 1;'
 }
 {
  engine: 'mysql',
  field: 'image { src }',
  query: 'SELECT src FROM images WHERE product_id = 2;'
 }
 {
  engine: 'mysql',
  field: 'image { src }',
  query: 'SELECT src FROM images WHERE product_id = 3;'
 }
]
```

  </td>
  </tr>
</table>

## Timetable / Outline

### Deadlines

September 19th 2017 - First draft of proposal due

December 8th 2017 - Progress report

January 29th 2018 - Oral presentations

March 10th 2018 - First draft of final report

March 18th 2018 - Poster Fair

### Milestones

October 1st - MVP, understanding of how graphql-ruby works, identify maintainers vision for the query planning feature

November 1st - N+1 identification

December 1st - Relational Database Integrations

February 1st - ElasticSearch Integration

March 1st - Upstream PRs Merged / Standalone Gem Published
Required Components / Facilities

Potential need for server hosting for testing integration with an application in a production environment. References for different datastores may be needed so we can adequately assess what kind of data should be available about their queries.

## References

[1] https://github.com/rmosolgo/graphql-ruby/issues/732

[2] https://github.com/Shopify/graphql-batch

[3] https://facebook.github.io/graphql
