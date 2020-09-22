# Guidelines Next Front-end

- [Guidelines Next Front-end](#guidelines-next-front-end)
- [1. Next JS Packages](#1-next-js-packages)
- [2. Naming conventions](#2-naming-conventions)
- [3. Creating a new functional Components for Pages and reusable component.](#3-creating-a-new-functional-components-for-pages-and-reusable-component)
    - [Functional Component](#functional-component)
    - [So why should I use functional components at all?](#so-why-should-i-use-functional-components-at-all)
- [4. Routes](#4-routes)
- [5. Don't repeat yourself (DRY)](#5-dont-repeat-yourself-dry)
- [6. General rules](#6-general-rules)
- [7. Reference](#7-reference)

# 1. Next JS Packages

- [next-routes](https://github.com/fridays/next-routes) (For routing pages)
- [express](https://expressjs.com/) (For serving the pages server side)
- [next-i18next](https://github.com/isaachinman/next-i18next) (Handles translation support)
- [react-hook-form](https://react-hook-form.com/) (Form validations)

# 2. Naming conventions

| What                 | How                  | Implementation | Location   | Other name rule                                                                                                                                                                                   |
| -------------------- | -------------------- | -------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Component            | PascalCase, singular | components     | components | Component should be descriptive on what the components functionalities. It should be reusable.                                                                                                    |
| Pages                | camelCase, singular  | pages          | pages      | Pages that is created should be group on related pages for organization, file name should be related on what the page renders.                                                                    |
| Variable declaration | camelCase            | -              | -          | the use of const and let should be strictly implemented for ES6 standard, variable should be understandable on it's purpose.                                                                      |
| Function Declaration | camelCase            | -              | -          | Functions that is created should be understandable on it's purpose and it should have maximum of 3 parameters and should only do one job for code reusability and so that it can be debug easily. |

# 3. Creating a new functional Components for Pages and reusable component.

Utilized the used of react hooks, (useState, useEffect)

### Functional Component

```javascript
  /**
    This component is for Footer consist of links and etc.
  */
  const Footer = props => {
    return (
      <div>
        <h1>This is footer</h1>
      <div>
    )
  }

  export default Footer;
```

### So why should I use functional components at all?

Benefits you get by using functional components in React.

- easier to read and test
- less code
- best practices.
- performance

[Please see reference for better understanding.]("https://medium.com/@Zwenza/functional-vs-class-components-in-react-231e3fbd7108")

# 4. Routes

All routes should be set inside the routes.js and server.js.

Example code inside routes.js

```javascript
const routes = require("next-routes");
const config = require("next/config").default();
let routePrefix = "";

if (config.routePrefix) routePrefix = config.routePrefix;

module.exports = routes()
  .add("index", `${routePrefix}/`, "/")
  .add("about", `${routePrefix}/about`, "/about");
```

If you can see the code about it has a variable for routePrefix it is set from .env file. We do it like this because when deploying to staging server we use /name-of-project in our domain names that is why we need to prepare the app for different environments depening on the values inside .env file we set.

# 5. Don't repeat yourself (DRY)

Components and functions inside libraries should have one purpose and responsibility, to lessen bugs and so that we can easily debug the code without affecting other parts of the app.

# 6. General rules

- Proper indentions of (2 spaces)
- Proper singular/plural naming
- Remove unused codes like variables, deprecated functions etc.
- Add comments that describes the functionalites of component or functions you will create.
- File/class names should be DESCRIPTIVE
- Method names should be DESCRIPTIVE
- DO NOT put logical codes inside pages. Create libraries and actions for specific task in your code.
- Make sure that the next person that will read your code can understand it without you asking.

# 7. Reference

- [https://medium.com/yals/dry-out-react-code-into-presentational-components-8308f42a8b80](https://medium.com/yals/dry-out-react-code-into-presentational-components-8308f42a8b80)
- [https://medium.com/@Zwenza/functional-vs-class-components-in-react-231e3fbd7108](https://medium.com/@Zwenza/functional-vs-class-components-in-react-231e3fbd7108)
