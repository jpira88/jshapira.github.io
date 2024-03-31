---
layout: post
title:  "Creating and sharing private npm packages without setting up a repository manager, using git urls" 
date:  2022-07-10 18:31:29 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["Architecture", "FrontEnd", "BackEnd", "Devops"]
---

* TOC
{:toc}


### Intro
When working on JS/TS components (or any other), most chances are, that on relatively regular basis, you're required to reuse some code you or your team wrote in multiple projects. 

A project can be multiple components of the same product, or completely different products.  
A common approach to conveniently share your code is wrapping it as a package and import it wherever required. 
But, due to various considerations (security, copyrights, etc'), you may want to refrain from uploading your code to NPM registry.  

In this case, you might set up a private repository manager such as Nexus or Verdaccio.  
But if your only goal is to share packages between your team, setting up and maintaining a server might be a bit of an overkill.  

Luckily, we can import code as NPM package directly from a private (or public) git repository.

For example, 

{% highlight json %}
{
  "name": "your-project",
  "version": "0.0.0",
  "scripts": {
    ...
  },
  "dependencies": {
    ...
    "your-package": "git+https://your-private-git-repo-url/your-package.git#semver:^x.x",
    ...
  },
  "devDependencies": {
    ...
  }
}

{% endhighlight %}

In the example above, your shared code will be available as a regular npm package named <i>your-package</i> .

{% highlight javascript %}
import {YourClass} from 'your-package';
{% endhighlight %}

#### Package Versioning
We can control which version of our shared package we install in multiple ways:
1. <i>#semver:^x.x</i> - npm will look for any tags or refs matching that range in the remote repository
2. <i>#branch-name</i> - npm will take a direct branch name
3. <i>#commit</i> - npm will take a specific commit

#### Caveats for npm packages consumed directly from git
An NPM package following best practices will usually have magic scripts such as:
1. prepare - usually will run a build script
2. prepublishOnly - will usually run tests and linters
3. preversion - will usually run a linter
4. version - can automatically create commits on version bumps
5. postversion - can automatically push to git 

We can't have this in a package consumed directly from git, since there is no publish process.

Another disadvantage will be the size of the package. 
In a regular package we publish the compiled code, but don't commit it to a source control.
Meaning we have less code in git and consumers of our package will not receive the source code.

In a git based npm packaged we must commit the compiled code together with the source code, otherwise our main project
will not be able to use the package.

#### Tree shaking to mitigate the caveats
For inconvenience in the lack of magic commands there's no straightforward solution (at least not one I'm aware of).  
But for the size issue, we have tree shaking.
If you're not familiar with the term, tree-shaking is a step in build process that removes unused code, thus reducing the size of the final application.
If you're not using extremely outdated frameworks and build tools, you'll usually get it with tools which come bundled with the framework.

This means that even if you install a git based npm package, the unnecessary source code will not be part of your compiled production application.


#### Wrapping your code as installable npm package
Initialize NPM package:
{% highlight bash %}
npm init -y
{% endhighlight %}
 

Add typescript as a development dependency:
{% highlight bash %}
npm install --save-dev typescript
{% endhighlight %}
 

Create tsconfig.json:
{% highlight bash %}
{
  "compilerOptions": {
    "target": "es5",
    "module": "commonjs",
    "declaration": true,
    "outDir": "./lib",
    "strict": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "**/__tests__/*"]
}
{% endhighlight %}


I'm not going to go into tsconfig.json options since it's out of context.  
You can find a <a href="https://www.typescriptlang.org/tsconfig" target="_blank">comprehensive documentation</a> in the official website.

#### Expanding the default package.json
The final basic version of the package.json should look like this:

{% highlight json %}
{
  "name": "your-package",
  "version": "1.0.0",
  "description": "You package description",
  "main": "lib/index.js",
  "types": "lib/index.d.ts",
  "scripts": {
    "build": "tsc"
  },
  "dependencies": {
    ...dependencies of your package
  },
  "devDependencies": {
    ...
    "typescript": "^4.0.5"
    ...
  },
  "repository": {
    "type": "git",
    "url": "https://your-private-git-repo-url/your-package.git"
  },
  "type": "commonjs",
  "author": "Your-name",
  "license": "ISC"
}
{% endhighlight %}

The main keys here are <i>main</i> and <i>types</i> .

- <i>main<i/> - indicates to npm from where to import the modules
- <i>types</i> - indicates for typescript and IDEs from where they can get type definitions for better developer experience

### Hello World
Once we got our basics ready, we can create a simple hello world package.

Let's create a file: <i>src/hello-world.ts<i>

{% highlight javascript %}
export class HelloWorld {
    public greet = () => 'Hello World';
}
{% endhighlight %}

Now we need to make it exportable, create a file <i>src/index.ts</i>
{% highlight javascript %}
export {HelloWorld} from './hello-world';
{% endhighlight %}


Next, lets build it:
{% highlight bash %}
npm run build
{% endhighlight %}

After this, you'll see new files added to the <i>/lib</i> folder of your project.

#### Pushing to git
For simplicity let's use a simple branch name and not semvers, name your branch: <i>my-first-release</i>, and push the code.

#### Using the package from the main project
Let's install the package:
{% highlight bash %}
npm install git+https://your-private-git-repo-url/your-package.git#my-first-release --save
{% endhighlight %}

Now we can use it from the main project:
{% highlight javascript %}
import {HelloWorld} from 'your-package';
{% endhighlight %}