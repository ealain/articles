---
published: true
title: 'Clean authorization control in serverless functions'
cover_image: 'https://dev-to-uploads.s3.amazonaws.com/uploads/articles/m686i5ynnovkacvth2pz.png'
description: 'Learn to properly structure your serverless code to achieve perfect authorization through a simple example use case'
tags: security, AWS, lambda, serverless
canonical_url:
---

In this article, I walk you through some control points to clean up authorization code in your serverless functions.

Serverless applications have a lot of benefits. They are easy to deploy and scale. They are also cheap to run. 🤑 However, they come with a few drawbacks. One of them being authorization hard to get right. On this matter is easy to make mistakes, write poorly maintainable code and introduce vulnerabilities. 😱

# 🕷️ Access control code can be ugly

In a typical **multitenant serverless application** on AWS, where Lambda is integrated with API Gateway for each endpoint, you **have to write authorization code**. This is code that will check whether the user who originated the request is allowed to access data or perform an action.

While in a monolithic application, you can rely on a framework to help you with that, **you are on your own with serverless functions**.

As your application evolves and becomes more complex, this becomes hard to maintain and check for vulnerabilities.

Hopefully, you can come up with a good access control strategy if you properly **structure your code**. 🏗️

# 🤦‍♂️ Naive solution: authorization code in the business logic

Let's consider a simple multitenant application to manage files for organizations. The application has two personas: CEO and user. CEO can access all files within their organization. Users can only access files they have created.

A **poorly designed function** that handles the request to get a file might look like this:

```typescript
// POORLY designed function

export const handler = async (event, context) => {
  // Check that the user is in the requested organization
  // 🐍 authorization with business logic not directly related to the use case
  const userId = context.requestContext.authorizer.userId;
  const organizationId = event.pathParameters.organizationId;
  const user = await getUser(userId);
  if (!user.organizations.includes(organizationId)) {
    return {
      statusCode: 403,
    };
  }

  // Retrieve the file
  // 🚀 business logic
  const fileId = event.pathParameters.id;
  const file = await getFile(fileId);

  // Deny access if the user is not CEO or the author is not the user
  // 🐍 access control happening late
  if (event.requestContext.authorizer.role !== 'CEO' && file.authorId !== event.requestContext.authorizer.userId) {
    return {
      statusCode: 403,
    };
  }

  return {
    statusCode: 200,
    body: JSON.stringify(file),
  };
};
```

# 🧅 Share authorization code that all your functions use

In many cases, you will have to write the same authorization code in multiple functions. For example, you might want to check that the user is in the requested organization. You can **share this code in a middleware**. If you are using AWS Lambda, you can rely on [middy](https://middy.js.org/).

Using a middleware will separate authorization code from the business logic and make it much **easier to read**.

Organizing your code in layers helps a lot. For example, you can enrich the context with the user information in a middleware, before authorization, for later use in subsequent steps:

```typescript
// Add the user to the context to use this information later in the authorization step
// src/middlewares/addUserToContext.ts

export const addUserToContext = async (event, context) => {
  const userId = event.requestContext.authorizer.userId;
  const user = await getUser(userId);
  context.user = user;
};
```

# ⏱️ Early check authorization

Denying a rogue access as early as possible is a good practice.

Frameworks like [Spring](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html) and [Nest](https://docs.nestjs.com/security/authorization) have **decorators** that you can use as soon as your function definition.

Find a way to **check authorization as early as possible** in the function.

```typescript
// src/handlers/getFile.ts

const getFile = async (event, context) => {
  // Check that the user is in the requested organization
  if (!context.user.organizations.includes(organizationId)) {
    return {
      statusCode: 403,
    };
  }

  // If the user is not CEO, add a filter to only retrieve the files created by the user
  const filters = {};
  if (event.requestContext.authorizer.role !== 'CEO') {
    filters.authorId = event.requestContext.authorizer.userId;
  }

  // Retrieve the file
  const fileId = event.pathParameters.id;
  const file = await getFile(fileId, filters);

  // Deny access if the user is not CEO or the author is not the user
  if (file === null) {
    return {
      statusCode: 404,
    };
  }

  return {
    statusCode: 200,
    body: JSON.stringify(file),
  };
};

export const handler = middy(getFile).use(addUserToContext);
```

Again, leverage middleware to extract the authorization code from the business logic. ⛏️ See the next section for a full example.

# 📁 Use file hierarchy to identify shared and specific code

Mind the **file hierarchy**. You should identify shared and specific code at a glance 👀 Below you can see that every handler might have a specific `validateAccess` logic, while shared middleware is available higher in the hierarchy.

```
src/
  handlers/
    getFile/
      getFile.ts
      index.ts
      validateAccess.ts
  middlewares/
    addUserToContext/
      addUserToContext.ts
      index.ts
    validateTenancy/
      validateTenancy.ts
      index.ts
```

The **function now only contains business logic**.

```typescript
// src/handlers/getFile/getFile.ts
export const getFile = async (event, context) => {
  // Retrieve the file
  const file = await getFile(filters);

  if (file === null) {
    return {
      statusCode: 404,
    };
  }

  return {
    statusCode: 200,
    body: JSON.stringify(file),
  };
};
```

Middleware can be found at the handler definition level.

```typescript
// src/handlers/getFile/index.ts
export const handler = middy(getFile).use(addUserToContext).use(validateTenancy).use(validateAccess);
```

Shared middleware to validate tenancy (user is requesting a resource in their organization).

```typescript
// src/middlewares/validateTenancy.ts
const validateTenancy = async (event, context) => {
  // Check that the user is in the requested organization
  if (!context.user.organizations.includes(organizationId)) {
    return {
      statusCode: 403,
    };
  }
};
```

Specific middleware to validate access (user is requesting a resource they have access to).

```typescript
// src/handlers/getFile/validateAccess.ts
const validateAccess = async (event, context) => {
  if (event.requestContext.authorizer.role === 'CEO') {
    return;
  }

  event.filters = {
    fileId: event.pathParameters.id,
    authorId: event.requestContext.authorizer.userId,
  };
};
```

Pro tip 🚀 prefer **allow list based authorization** over deny list based authorization. This is a best practice, that will deny access by default.

# Useful links

- [AWS RESTful API](https://aws.amazon.com/what-is/restful-api/)
- AWS Lambda middleware tooling: [middy](https://middy.js.org/)
- With a proper code structure, you can easily evolve to [API Gateway Lambda authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)
