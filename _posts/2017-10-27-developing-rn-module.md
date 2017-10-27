---
title: Developing a module for React Native
description: Get a local module working with a React Native app for quick development
image: /assets/images/react-native-module.jpg
layout: post
---

After building an app using React Native and using modules from NPM, you may decide that a component in your app could be used by others and you want to share it to NPM. While the typical "How to deploy a JS module to NPM" will get you most of the way there, React Native has a few caveats that need to be addressed.

Lets start by finding a component in our app that can be pulled out into it's own module. I'm going to do a simple one just to make things easy, a text component that will add ellipsis on the end of a line

{% highlight javascript %}
import { Text } from 'react-native'

export default (props) => {
  return (
    <Text {...props} numberOfLines={1} />
  )
}

{% endhighlight %}

The first thing we need to do is create a new directory where our module repository will live. After that we need to initialize that directory as an npm module. To do this you run `npm init` inside the directory. This will walk you through several questions such as the name of the module (this must be unique across all of npm), the version number, and the license. When it asks you about the entry point leave it as `index.js`. At the end of this process you will have a file called `package.json`.

{% highlight javascript %}
{
  "name": "react-native-text-ellipsis",
  "version": "1.0.0",
  "description": "A react native component that will truncate text to one line and add an ellipsis",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT"
}
{% endhighlight %}

The main field describes what file should be used when someone imports your module. In our case when someone imports `react-native-text-ellipsis` they will get our `index.js`. So `index.js` is where we should put our component.

![index.js of our module](/assets/images/react-native-module-index.jpg)

Now that we have our `index.js` we need to set up the dependencies this module relies on. In our `index.js` file we are importing `react-native` so we will need that. We will also need `react` because we are using JSX. The special case with these two modules is that while our module needs them, the app that is consuming our module will already have them. If we were to install `react-native` and `react` as dependencies of our module, the consumer of our module might end up with multiple copies of `react` and `react-native` which isn't good. NPM provides a solution for this problem in the way of `peerDependencies`. `peerDependencies` are a way for an npm module to inform consumers of what dependencies are required in order to use the module. It is then expected that the consumer of your module will install your `peerDependencies` as their own project depedencies. `peerDependencies` do not get installed into your module when you run `npm install` since it's expected that the consumer will have the dependencies.

Unfortunately the npm cli doesn't have a way to add `peerDependencies` to your `package.json`, which means we will need to edit it by hand.  When adding a peer dependency it is possible to indicate what version of the dependency your module needs in order to function properly. While you could use `*` to indicate all versions, if your module truly depends on features only in a specific version of the dependency you should be explicit.

{% highlight javascript %}
{
  "name": "react-native-text-ellipsis",
  "version": "1.0.0",
  "description": "A react native component that will truncate text to one line and add an ellipsis",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "MIT",
  "peerDependencies": {
    "react": ">= 15.x",
    "react-native": ">= 0.47.x"
  }
}
{% endhighlight %}

The last thing that we need to do is to hook it up to babel. Because the React Native packager already runs babel and will transpile your module when it packages up the consumers app, we don't need to install babel in this module. But we do need to tell the consuming app's packager how we want babel to work with our module. First off we need to install the react native babel preset into our module as a dev dependency since we don't need babel in a production version of our app. To do so run `npm install -D babel-preset-react-native`. After that installs we need to create a `.babelrc` file at the root of our module, the contents of which are as follows.

{% highlight javascript %}
{
  "presets": ["react-native"]
}
{% endhighlight %}

At this point we could publish this module to npm by running `npm publish`. Then in our app we can `npm install <module_name>` and use our component. If you want to see how the module works before publishing you can link the module into your app on your local filesystem. From the directory of your module run `npm link`. This will tell npm that you intend on linking to this module from some other project on your machine. Then navigate to your app project directory and run `npm link <module_name>`. This will create a symlink in your `node_modules` directory for your app that points to the module on your local filesystem. You are then free to make changes in your module and should see those changes immediately in the app.

But our module should really be tested, which I won't go over here, but the need for testing introduces some weirdness in terms of publishing to npm and consuming a react native module. In order to test our component we are going to need both `react` and `react-native` as dependencies for our tests to run. We will install these as `devDependencies` since `peerDependencies` don't get installed in a project and we only need these dependencies for testing. You can do this by running `npm install -D react react-native`.

This introduces a problem with linking to a module on your local filesystem though. Lets say we run `npm install` inside our module. This will install `react`, `react-native`, and `babel-preset-react-native`. We then run `npm link`. Then we go to our app and run `npm link <module_name>`. So far everything works. We go to start the packager and we get a big error complaining that two modules `@providesModule` the same name and that there must be duplicate versions of `react-native`. That is because our app has a version of `react-native` and now our module has a version of `react-native` which both have special syntax in them specfic for the react native packager. The easiest way I've found to get around this problem is to go back into your module directory and run `rm -rf ./node_modules/react-native`. This will allow you to use your module in your app, but if you need to run your tests again you will need to do another `npm install` to get `react-native` back for your tests.