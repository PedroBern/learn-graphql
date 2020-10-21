---
title: "Set up a GraphQL client with Relay"
metaTitle: "Relay Compiler GraphQL Setup | GraphQL React Relay Typescript Tutorial"
metaDescription: "You will learn how to configure Relay in React by installing dependencies like react-relay, relay-config, babel-plugin-relay, relay-compiler"
---

import GithubLink from "../src/GithubLink.js";

Relay gives a neat abstraction layer and an interface to your GraphQL server. You don't need to worry about constructing your queries with request body, headers and options, that you might have done with `axios` or `fetch` say. You can directly write queries and mutations in GraphQL and they will automatically be sent to your server via your Relay Environment instance.

### Relay & Relay Compiler Installation

Let's get started by installing relay & dev setup dependencies:

```bash
$ yarn add react-relay
$ yarn add --dev relay-config babel-plugin-relay graphql relay-compiler
$ yarn add --dev @types/react-relay @types/relay-runtime
```

Relay compiles the graphql queries ahead-of-time with the `relay-compiler` script. The best way to run it is with a yarn/npm script by adding a script to your package.json file.

Open `package.json` and add the following:

```json
"scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
-   "eject": "react-scripts eject"
+   "eject": "react-scripts eject",
+   "relay": "relay-compiler"
  },
```

But still needs to pass the configuration, this is where the `relay-config` package comes in, it provides a way of having a single configuration file for both `babel-plugin-relay` and `relay-compiler`.

Open `relay.config.js` and add the following:

```js
module.exports = {
	src: "./src",
	schema: "./schema.graphql",
	exclude: ["**/node_modules/**", "**/__generated__/**"],
	extensions: ["ts", "tsx"],
	language: "typescript",
	artifactDirectory: "src/__generated__/relay",
	customScalars: {
		timestamptz: "string",
	},
};
```

<!-- TODO: check if it is really necessarie, maybe the graphql import from relay works -->

As the boilerplate was created with the `create-react-app`, it already works with the `babel-plugin-relay`
out of the box, we just need to make sure to import the `graphql` like below:

```js
import graphql from "babel-plugin-relay/macro";
// instead of:
// import { graphql } from "babel-plugin-relay"
```

### Create the Relay Environment

Open `src/components/App.tsx` and add the following imports at the top:

<GithubLink link="https://github.com/hasura/learn-graphql/blob/master/tutorials/frontend/typescript-react-relay/app-final/src/components/App.tsx" text="src/components/App.tsx" />

```javascript
import * as React from 'react';

+ import {
+   Environment,
+   Network,
+   RecordSource,
+   Store,
+   FetchFunction
+ } from 'relay-runtime';

import Header from './Header';
import TodoPrivateWrapper from './Todo/TodoPrivateWrapper';
import TodoPublicWrapper from './Todo/TodoPublicWrapper';
import OnlineUsersWrapper from './OnlineUsers/OnlineUsersWrapper';

import createContext from "../utils/createContext"

import { useAuth0 } from "./Auth/react-auth0-spa";

const App = ({idToken}:{idToken:string}) => {
  const { loading, logout  } = useAuth0();
  if(loading) {
    return (<div>Loading...</div>);
  }
  return (
    <div>
      <Header logoutHandler={logout} />
      <div className="container-fluid p-left-right-0">
        <div className="col-xs-12 col-md-9 p-left-right-0">
          <div className="col-xs-12 col-md-6 sliderMenu p-30">
            <TodoPrivateWrapper />
          </div>
          <div className="col-xs-12 col-md-6 sliderMenu p-30 bg-gray border-right">
            <TodoPublicWrapper />
          </div>
        </div>
        <div className="col-xs-12 col-md-3 p-left-right-0">
          <div className="col-xs-12 col-md-12 sliderMenu p-30 bg-gray">
            <OnlineUsersWrapper />
          </div>
        </div>
      </div>
    </div>
  );
};

export default App;
```

These are the required relay dependencies to get started. Now let's define a function creator for a function which will execute the queries, and a context provider for the Relay Environment:

```javascript
type Props = {
  idToken: string
};

+ const buildFetchQuery = (authToken: string): FetchFunction =>
+   async (operation, variables) => {
+     // TODO: get the correct url
+     return fetch('https://hasura.io/learn/relay', {
+       method: 'POST',
+       headers: {
+         'Content-Type': 'application/json',
+         'Authorization': `Bearer ${authToken}`
+       },
+       body: JSON.stringify({
+         query: operation.text,
+         variables,
+       }),
+     }).then(response => {
+       return response.json();
+     });
+   };
+
+ const [useEnviroment, EnvironmentProvider] = createContext<Environment>();
+
+ const RelayProvider:React.FC<Props> = ({
+   children,
+   idToken
+ }): React.ReactElement => {
+   const [environment] = React.useState(
+     new Environment({
+       network: Network.create(buildFetchQuery(idToken)),
+       store: new Store(new RecordSource()),
+   }));
+
+   return (
+     <EnvironmentProvider value={environment}>
+       {children}
+     </EnvironmentProvider>
+   )
+ };
```

Inside the `App`, pass the idToken prop to `<RelayProvider>` component.

```javascript
const App:React.FC<Props> = ({ idToken }) => {
  const { loading, logout  } = useAuth0();

  if(loading) {
    return (<div>Loading...</div>);
  }
  return (
+   <RelayProvider idToken={idToken}>
      <div>
        ...
      </div>
+   </RelayProvider>
  );
};

export default App;

+ export { useEnviroment };
```

Let's try to understand what is happening here.

### Relay Environment

TODO
