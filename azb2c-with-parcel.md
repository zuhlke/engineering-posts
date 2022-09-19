---
title: Build Azure AD B2C Templates
subtitle: A simplistic approach using parcel.js
domain: campzulu.hashnode.dev
tags: web-development, azure, az-b2c, sass
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1663572757122/uN-z_ruLg.jpg?auto=compress
publishAs: csalv
ignorePost: true
---



# The goal
When integrating Azure Active Directory B2C service into your app, you can customize the ![HTML with templates](https://docs.microsoft.com/en-us/azure/active-directory-b2c/customize-ui-with-html) to provide a more immersive user experience. This article will describe a simple approach on how to setup a local project to develop such templates and create a production ready build.

The basics to do so are simple enough: Publish a public available HTML page with styles/js to your liking that includes the following tag, can be used by the Azure B2C engine to render its login input fields inside:

```html
<div id="api"></div>
```

But immediately some challenges came to mind: How do we get a production ready optimized build (css transpiling, minifying, ...) as well as some improved developer experience (scss preprocessor)

# The idea

This question lead us to ![parcel.js](https://parceljs.org/), as we wanted to have something simple and with as little configuration as needed.

You can jump into the ![github repo](https://github.com/csalv22/az-b2c-parcel/) directly to have a look implementation.

# Setup the code
Lets get our hands dirty and create add a the dependency to the current project (or create a new project with ```yarn init```) ```yarn add --dev parcel``` 

Create a new ```/src/``` directory and add a new file ```en.html```:
```html
<!DOCTYPE html>
<html>
<head>
    <title>My Product Brand Name</title>
</head>
<body>
    <div id="api"></div>
</body>
</html>
```

In the ```package.json``` add a new line ```"start-b2c": "parcel ./src/*.html", ``` in the scripts section.
Now start up your application via console ```yarn start-b2c```  and access the template on  [http://localhost:1234/en.html](http://localhost:1234/en.html)

Congratulations, you have a basic template that would already be working.


## HTML
Now the HTML above can be extended with additional structures to fulfill the design as desired. A [sample version](https://github.com/csalv22/az-b2c-parcel/blob/main/src/en.html) with a bit more features can be found, but writing html & scss is not what this blog post is about.

## CSS
Now to the interesting part: Styling the new template we have a few things to take care off until we can properly design our page.

### SASS / SCSS
We will rely on [sass](https://sass-lang.com/) as preprocessor to have a stronger toolset at hand than just plain css. This can very easily be [added to our setup](https://parceljs.org/languages/sass/): ```yarn add --dev @parcel/transformer-sass```

Create a new stylesheet in ```/src/scss/index.scss``` with the following content
```scss
div {
  &#api {
    width: 100px;
    height: 100px;
    background-color: #123456;
  }
}
```

Rerunning your developer process should already apply the SCSS style you created.
Now your set up to design your html/css structure and immediately verify the result in the browser.

### CSS Normalize / Reset
As still all browsers render unstyled html elements slightly differently, the current web site would result in different appearance on the different browsers, which we have to address.
We decided to go for [normalize.css](github.com/necolas/normalize.css) as a hard reset felt like an overkill.

### CSS Transpiling
[Transpiling](https://parceljs.org/languages/css/#transpilation) is done out of the box by parcel.js, so if you have an appropriate configuration in your package.json, no additional steps are required.

## JS
For our usecase we dont have the need to have any custom JS, plain html & css serves our needs. But parcel.js gives you many options into hand (you could even go for one of the big JS Frameworks and use static site rendering to create your html templates)

## Styling and Testing the login form content
This far we built a lovely template, but have no idea how the form fields (input for login, pw reset, ...) can be styled. To do that, we need to test our template with the content from the azure b2c flows (and every configuration might have different content).
For that, access your current available b2c pages and on different screens use the html elements to replace ```<div id="api">...</div>``` to your local en.html. This way you can preview&design those pages before deployment.
We added a set of those content html elements in separate [*.html files](https://github.com/csalv22/az-b2c-parcel/tree/main/src/api-snippets_de) to quickly retest different scenarios (i.e. styling of error messages).

# Build
To get a production ready build is fairly simple: Add ```build-b2c: parcel build src/*.html --dist-dir ./public``` to your scripts section in the package.json and test it with ```yarn build-b2c```.

But hold on, if you were to deploy that now and integrate it with azure b2c, this wouldnt work: The azure b2c template mechanism has a restriction, in that it can't handle relative paths from your template like ```<link rel="stylesheet" href="en.6a135913.css"/>``` - it requires absolute paths. So you will need to define the public-url in your build step: ```build-b2c: parcel build src/*.html --dist-dir ./public --public-url 'https://mystorageaccount.blob.core.windows.net/az-b2c'```

The build output can be deployed manually to an [azure blob storage account](https://docs.microsoft.com/en-us/azure/active-directory-b2c/customize-ui-with-html?pivots=b2c-user-flow#2-create-an-azure-blob-storage-account) (or other hosting solution). We deployed it as part of our SPA in an azure static site (behind an app gateway) and integrated it with our CI/CD workflow.

## Configuration of Azure B2C
The configuration of the Azure B2C to use our templates was handled by a partner company (there are ways to configure it in the identity experience framework for all pages user flows, which is not obvious from microsofts documentation, so be prepared to dive into some xml configuration).

# Conclusion
Parcel.js provided us a quick and easy way to build those templates, we have now very clean and lightweight templates with neither much overhead on the development side nor on the output.
There are of course many steps we can take to improve our solution:
* sprinkle some JS to give a better result to the client (but best do so in a progressive manner)
* deliver [optimized images](https://parceljs.org/recipes/image/) 
* look for some templating mechanism for developer mode to test the different form content
* add a more versatile way for managing translations (and not duplicate the html structure for every language)