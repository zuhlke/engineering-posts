---
title: MicroFrontend - Module Federation with Vite for React
domain: software-engineering-corner.hashnode.dev
tags: web-development, microfrontend, module-federation, react, vite
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1681956795984/TRA8XJ_Ir.webp?auto=format
publishAs: NipunaMarcus
hideFromHashnodeCommunity: false
ignorePost: true
---

With the introduction to the [Microservice](https://martinfowler.com/articles/microservices.html) architecture for backend developers were looking into developing a similar architecture for frontend applications which is what we now known as [Micro frontends](https://micro-frontends.org/). 
This architecture has similar set of pros and cons as Microservices but for frontend. One of the implementations to achieve this architecture from bundling tool such as Webpack and Vite are called Module Federation.

## What is Module Federation?

Module federation is concept which get different builds to come together to make one application. So in most cases one will be the host application which will bring all other remote components that are built to be shared.

![How federation looks like](https://cdn.hashnode.com/res/hashnode/image/upload/v1681957205265/bcuUQoe-I.webp?auto=format)

Figure 1 shows rough explanation on how it works. In host application there will be a reference to each remote component as an import or a Lazy Loaded module. 
At the remote servers where each “HomePage App” and “Payment App” hosted, shared components are packaged into a javascript module and will be publicly available as a built javascript files (in above example “homepage.js” and “payment.js”). 
Then from host application side these remote components will be brought in at runtime as javascript module and will be treated as components in hosted application.

More details can be found [here](https://webpack.js.org/concepts/module-federation/) about concept of Module Federation.

## How to use module federation in Vite?

To get the module federation functionality we need to add a plugin to Vite. [Plugin can be found here](https://github.com/originjs/vite-plugin-federation).

There are always minimum two places where we need to configure to get the module federation functionality. One configuration should be applied on the remote application where we have the components to be shared to tell Vite that what are the components that will be shared as modules and what is the entry name for the build.
Other Configuration should be applied on the host application side where we gonna use the federated modules.

Following are the configurations to be applied on both sides.

## Remote Application Basic Configuration

```typescript
export default defineConfig({
    plugins: [
        react(),
        federation({
             name: 'remotecomponent1',
             filename: 'remotecomponent1.js',
             exposes: {
                 './Button': './src/components/buttons.tsx'
             },
             shared: ['react', 'react-dom', 'react-router-dom'],
        }),
    ]
});

```

Let’s go through each of the fields in federation plugin.

**Name**: is the module name to be given to the javascript module that will be built including shared components. This is required. More details can be found [here](https://github.com/originjs/vite-plugin-federation#name-string).

**FileName**: is the Filename of the entry file for the javascript module. This is not required though as the default value for this will be `remoteEntry.js`. More details can be found [here](https://github.com/originjs/vite-plugin-federation#filenamestring).

**Exposes**: is where we need to list down the components that we are going to expose as remote component with the remote module. More details can be found [here](https://github.com/originjs/vite-plugin-federation#exposes).

**NOTE**: All the exposing components should export the react component as a default export. Otherwise it will not be able to integrate without an issue from the Host application side as there is no enough details to import individual export from a react component.

**Shared**: So this is bit of a complex property. When it comes to libraries like react we need to have one instance shared between all the libraries to handle states of components without an issue. So when we are working with remote modules we need a way to use one react instance in both remote module and the host application. To achieve this we need tell that what libraries will be shared between host and remote modules. This property will enable us to list those libraries. So we need to add this property in both host side configuration and remote application side configuration and add the same list of libraries in both side to be indicate what need to be shared. More details on the property can be found [here](https://github.com/originjs/vite-plugin-federation#shared).

## Host Application Basic Configuration

There are two ways to configure host application to bring in remote modules. One is directly referring to the URL of the remote module entry file. Second is dynamically populate the reference. Both has a smilier set of properties as to the remote side except for the property remotes . As Nameand Shared property are same as explain above following is the explanation for remotes property.

Remotes: holds references to entry files of remote modules. Following are the examples of how we can use remotes property. More details can be found [here](https://github.com/originjs/vite-plugin-federation#remotes).

### Get the remote component using URL

```typescript
export default defineConfig({
    plugins: [
        react(),
        federation({
             name: 'remotecomponent1',
             remotes: {
                 sharedComp: 'http://localhost:3001/assets/remotecomponent1.js',
             },
             shared: ['react', 'react-dom', 'react-router-dom'],
        }),
    ]
});
```
### Get the Remote component by applying URL dynamically

```typescript
export default defineConfig({
    plugins: [
        react(),
        federation({
             name: 'remotecomponent1',
             remotes: [
                 {
                     sharedComp: {
                         external: `Promise.resolve(window.remoteURL)`,
                         from: 'vite',
                         externalType: 'promise',
                     },
                 },
             ],
             shared: ['react', 'react-dom', 'react-router-dom'],
        }),
    ]
});
```
**External**: Can be the address of the remote module, basically the URL or a Promise<string>. More details can be found [here](https://github.com/originjs/vite-plugin-federation#externalstringpromisestring).

**From**: will let the Vite know that from where the remote module coming from. Whether it’s coming from Webpack or Vite. More details can be found [here](https://github.com/originjs/vite-plugin-federation#from--vitewebpack).

**ExternalType**: will set what type of external reference will be used in external property. So value for this can be url or promise . More details can be found [here](https://github.com/originjs/vite-plugin-federation#externaltype-urlpromise).

**NOTE**: _here in the sample we are using `window.remoteURL` to get the url. So this is set from host application startup. so this property can be set on `app.tsx` or somewhere which is the root component of the application._

## How to use remote module inside the host application

There are two ways to use remote modules in a component in the host application.

### As a Static Import

We can always add the remote module as a static import inside a react component.

```typescript
import Button from 'sharedComp/Button';

function App() {
  return (
    <div className="App">
      <div className="card">
        <Button />
      </div>
    </div>
  )
}

export default App;
```
This is fine but as to the performance and reliability this is not the most promising way. I have faced few issues when comes to this method. Also using this its hard to handle network level issues.

### As a Lazy loaded module

This is to load the remote module using lazy loading to up the performance and network issue handling.

```typescript
import {lazy, Suspense} from 'react';
const Button = lazy(() => import('sharedComp/Button') as any);
function App() {
  return (
    <div className="App">
      <div className="card">
        <Suspense fallback={<div>Loading...</div>}>
          <Button />
        </Suspense>
      </div>
    </div>
  )
}

export default App;
```
This is my recommended way of using remote modules as this give us ability to handle network related issues and performance related issues.

**NOTE**: _if you prefer to implement this using typescript make sure you add a custom type declaration file to your src root and add the name of the remotes config as a module. file can be named something like `module.d.ts` ._

```typescript
declare module 'sharedComp/*' {}
```

## Running the applications

There are few things we need to keep in mind when we are running the both Host and Remote applications.

- Remember to run the remote application and host application in `Preview mode` when developing instead of `Development mode` to get the file serving working. Otherwise if we ran applications on dev mode because of dev server doesn’t serve files we cannot get the module federation working in dev mode.
- Also remember when a component is shared it will be shared as a javascript module and not as an application. The responsibility of an application will be carried out by the host application that will bring in and uses the shared module. Hence when remote component uses environment variable or any other process related data will be pushed through host application to remote components. So when using remote component remember that host application will always be the platform that this remote component running on. Because even though remote component running on a separate server remember that remote server only serving a file at this point when comes to module federation and nothing more. You still can see the remote server side application running using URL but from host application side we are only referencing to the shared javascript module entry `.js` file. So make sure to provide every process related data from Host application even though they are provided in you remote application side.

### Lets implement a Basic Scenario with what we learnt so far

Let’s consider an application use case which has different components which we can divide between different teams. Let’s take E-Commerce site as a scenario. Lets develop E-Commerce web app as host and then payment component and Home Page component. For Basic scenario lets implement only Home page use case for now.

I borrowed some code from [the vite-plugin-federation samples for react and Vite](https://github.com/originjs/vite-plugin-federation/tree/main/packages/examples/react-vite) to get the base application and created a [sample application](https://github.com/NipunaMarcus/module-federation) which showcase a basic use case of module federation.

In this sample, Home Page of our website is developed as a remote module which is referenced in the website (Host Application). Remote modules are imported as static imports in this sample ( Main branch ) and the remote module is reference using static URL.

```typescript
import { defineConfig } from 'vite';
import federation from '@originjs/vite-plugin-federation';
import dns from 'dns';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

dns.setDefaultResultOrder('verbatim')

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'app',
      remotes: {
        homepage: 'http://localhost:5000/assets/homepage.js',
      },
      shared: ['react']
    }),
    tsconfigPaths(),
  ],
  preview: {
    host: 'localhost',
    port: 5001,
    strictPort: true,
  },
  build: {
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  }
})
```
So from the remote application I have exposed two components, `Button` and `Home` , and added them to a module with the entry file name `homepage.js` .

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import federation from '@originjs/vite-plugin-federation';
import dns from 'dns';
import tsconfigPaths from 'vite-tsconfig-paths';

dns.setDefaultResultOrder('verbatim')

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'home',
      filename: 'homepage.js',
      exposes: {
        './Button': './src/components/button/index.tsx',
        './Home': './src/views/home/index.tsx'
      },
      shared: ['react']
    }),
    tsconfigPaths(),
  ],
  preview: {
    host: 'localhost',
    port: 5000,
    strictPort: true,
    headers: {
      "Access-Control-Allow-Origin": "*"
    }
  },
  build: {
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  }
})
```

Let’s look at the Home component which is shared from remote application.

```typescript
import { getProducts } from "../../utils/util";
import "./Home.css"

function Home() {
    // Get product names.
    const products: string[] = getProducts();
    return (
        <div className="home">
            <h3>Remote 1: Home Page</h3>
            {
                products.map((value: string) => {
                    return (
                        <div className="home-card">
                            <p>
                                {value}
                            </p>
                        </div>
                    )
                })
            }
        </div>
    );
}

export default Home;
```
Here if you closely look at the component you see that it is exported as default. **It is important to export the shared component as a default**. Without exporting the component as default, it will be hard to reference them from the Host application side.

From the Host application side when using the remote modules we can use static imports.

```typescript
import './App.css';

// Import remote modules using static imports.
// Use them inside the app component.
import Home from 'homepage/Home';
import Button from 'homepage/Button';

function App() {
  return (
    <div className="App">
      <h1>[Host Application] Welcome to the Shop!</h1>
      <div className='navBar'>
        <div>
          <Button caption="Home" />
        </div>
      </div>
      <div className="card">
        <Home />
        <p>
          Test App for host application.
        </p>
      </div>
      <p className="read-the-docs">
        Module federation with Vite and React
      </p>
    </div>
  )
}

export default App;
```

And after importing each of these component they can be used as any normal component. But remember using static import can cause issues when it comes to real production environments as these remote modules might not be available at the time. Hence using lazy loading on them will help when it comes to handling network related issues. You can find the lazy loading sample here in [this branch](https://github.com/NipunaMarcus/module-federation/tree/lazy-loading).

With lazy loading the host application side where we uses the remote modules are like below.

```typescript
import './App.css';
import React, { Suspense } from 'react';

// Import remote modules using lazy loading.
// Use them inside the app component.
const Home = React.lazy(() => import('homepage/Home'));
const Button = React.lazy(() => import('homepage/Button'));

function App() {
  return (
    <div className="App">
      <h1>[Host Application] Welcome to the Shop!</h1>
      <div className='navBar'>
        <div>
          <Suspense fallback={<div>Loading...</div>} >
            <Button caption="Home" />
          </Suspense>
        </div>
      </div>
      <div className="card">
        <Suspense fallback={<div>Loading...</div>} >
          <Home />
        </Suspense>
        <p>
          Test App for host application.
        </p>
      </div>
      <p className="read-the-docs">
        Module federation with Vite and React
      </p>
    </div>
  )
}

export default App;
```

In line 6 and 7 I have lazy loading the components from remote module. And then we are using that inside a `Suspense` component to load it in async and have a fall back component applied incase of loading takes time.

Also when importing these from remote modules if you are using typescript remember to add a custom definition file for types. So here in my implementation I have added the `custom.d.ts` file. This file help you to get rid of the compilation error `Module not found` .

```typescript
declare module 'homepage/*'
```
## Lets look in to implementing a bit advance scenario
Let’s add the payment component in to the story and make this a bit more complicated. Implementation for the advance scenario can be found in [this branch](https://github.com/NipunaMarcus/module-federation/tree/advance-use-case).

This adds bit of complexity as this uses two different remote modules to load home and payment pages and also added the complexity of react routing on to the host app. Also in this sample the payment page is loaded using a dynamically set URL which is pushed during the runtime through `window` object.

Vite configuration for this advance use case from host application side looks like this.

```typescript
import { defineConfig } from 'vite';
import federation from '@originjs/vite-plugin-federation';
import dns from 'dns';
import react from '@vitejs/plugin-react';
import tsconfigPaths from 'vite-tsconfig-paths';

dns.setDefaultResultOrder('verbatim')

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    react(),
    federation({
      name: 'app',
      remotes: [
        {
          homepage: {
            external: 'http://localhost:5000/assets/homepage.js',
            from: 'vite',
            externalType: 'url'
          },
        },
        {
          payment: {
            external: 'Promise.resolve(window.paymentUrl)',
            from: 'vite',
            externalType: 'promise'
          },
        }
      ],
      shared: ['react', 'react-dom', 'react-router-dom']
    }),
    tsconfigPaths(),
  ],
  preview: {
    host: 'localhost',
    port: 5001,
    strictPort: true,
  },
  build: {
    target: 'esnext',
    minify: false,
    cssCodeSplit: false
  }
})
```
Here in the config you can see I have added two different remotes configs to load the payment and home remote modules and two different external type for each. Here, as we did in the earlier examples, we are loading the home component with a static url which is embedded in to the config. But to load the payment module we are using the externalType, promise. I’m using this to demonstrate that incase I don’t know what will be the URL ,where Payment remote module is hosted on, in the build phase I should be able to set it in the runtime. This will be helpful if we are using and pipeline to host our applications and all the urls will be resolved at runtime. So as you can see in the line 25 I’m using below config to use a promise to load the payment remote module’s url during runtime.

```typescript
remotes: [
...
{
  payment: {
     external: 'Promise.resolve(window.paymentUrl)',
     from: 'vite',
     externalType: 'promise'
  },
}
...
```

And inside the `App.tsx` component inside the host application I have set the payment remote module url to the `window` object and lazy load the payment remote module addition to the home page remote module.

```typescript
import './App.css';
import React, { Suspense, useEffect } from 'react';
import { ExtendedWindow } from 'types';
import { BrowserRouter, Routes, Route, Navigate } from "react-router-dom";
import Navigation from './components/navigationBar';

// Import remote modules using lazy loading.
// Use them inside the app component.
const Home = React.lazy(() => import('homepage/Home'));
const Payment = React.lazy(() => import('payment/Payment'));

declare let window: ExtendedWindow;

function App() {
  
  useEffect(() => {
    // This is not as elegant as production code as we cannot use such in production.
    // But lets assume these envs are coming from a environment handler.
    // And this is only for sample case.
    if (import.meta.env.VITE_PAYMENT_URL) {
      // This is where we set the payment remote's URL.
      window.paymentUrl = import.meta.env.VITE_PAYMENT_URL;
    }
  }, []);

  return (
    <div className="App">
      <h1>[Host Application] Welcome to the Shop!</h1>
      <BrowserRouter>
        <Navigation />
        <div className="card">
          <Routes>
            <Route path="/" element={<Navigate to='/home' />} />
            <Route path='/home' element={
              <Suspense fallback={<div>Loading...</div>} >
                <Home />
              </Suspense>
            } />
            <Route path='/payment' element={
              <Suspense fallback={<div>Loading...</div>} >
                <Payment />
              </Suspense>
            } />
          </Routes>
          <p>
            Test App for host application.
          </p>
        </div>
      </BrowserRouter>
      <p className="read-the-docs">
        Module federation with Vite and React
      </p>
    </div>
  )
}

export default App;
```

## Possible errors faced during development and possible solutions.

1. **React, React Router Dom and other shared libraries becoming undefined.**

What you need to check is
  * You have not added the correct list of shared libraries to `shared` property on either your remote app Vite config or host app Vite config. Shared list of the both side should be same.
  * Check anything that should be shared between remotes and host app is added to the shared list.
2. **Failed to load the remote modules.**
 ![Error in similar case will look like this](https://cdn.hashnode.com/res/hashnode/image/upload/v1681972716537/H7Z_c3bgp.webp?auto=format)

What you need to check is
  * Check whether each remote module names in host side Vite config is unique.
  * Check whether correct component name is mentioned in the remote side Vite config. inside the config the component name should be similar to the component you are looking to share.
  * Check whether component is exported as default from the remote application side.
3. **If all the packages are in a monorepo structure there is an issue with npm hoisting and sharing libs between host and remote modules.**
   
[Building host side fails with hoisted dependencies in monorepo when using lerna](https://github.com/originjs/vite-plugin-federation/issues/276)

This is the opened issue for this in the vite-plugin-federation side. But if you need to keep the monorepo structure what you need to do is follow what i have done in the above sample projects, install node modules in each individual application instead of root of the workspace.

## Conclusion
* Always better to use lazy loading when it comes to importing remote modules instead of static import.
* Use static URL in config if you know the URL for remotely hosted modules and if need to resolve the URL dynamically at runtime then use promise method to resolve the URL.
* Remember to add everything to the shared component for it to work without a hitch. Examples: if there is a provider which you have applied in you remote application root level and you are using this inside the shared component remember to either add this provide inside the shared component or create a provider from host application side and wrap the remote component with that(haven’t tested this way though)… This is because once you share the component only that component and imports needed for it will be packed together to the remote module. So what you add in the root component of the remote app won’t be available inside the shared component.
* Always share a core libraries such as vue and react between the host and remotes.
