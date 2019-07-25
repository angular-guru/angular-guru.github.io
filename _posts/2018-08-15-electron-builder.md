---
layout: post
title:  "Create Electron Apps with Angular CLI"
description: "Build Electron apps with the Angular CLI using the Electron Build target!"
image: assets/images/electron-builder.png
slug: electron-builder
date:   2018-08-15
---


This is a builder for the Angular CLI that allows you to target an Electron environment, giving you access to all functions available to Electron such as file system access, which currently is not supported in the Angular CLI.

For example, you can do imports such as:

{% highlight typescript %}
{% raw %}
import { readFile } from 'fs';
{% endraw %}
{% endhighlight %}

It is important to note, this is not schematics or anything that will help scaffold an Electron application for you. This is simply an extension to the Angular CLI build step to allow your Angular app to have full access to Electron's features without any awkward workarounds like message passing.

### How To Use

The setup process is now incredibly simple. First install the package:

{% highlight bash %}
{% raw %}
npm install @angular-guru/electron-builder --save-dev
{% endraw %}
{% endhighlight %}

Next, we need to update `angular.json` to use the `electron-builder` in two places:

{% highlight typescript %}
{% raw %}
"build": {
    "builder": "@angular-guru/electron-builder:build",
    ...
}

"serve": {
    "builder": "@angular-guru/electron-builder:dev-server",
    ...
}
{% endraw %}
{% endhighlight %}

Your app should now able to take advantage of all the wonderful features Electron can offer us!

You can find a very basic [example project](https://github.com/angular-guru/electron-seed) for an example of how to get up and running!