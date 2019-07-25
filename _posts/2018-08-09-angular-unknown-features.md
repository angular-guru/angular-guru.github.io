---
layout: post
title:  "10 Useful Angular Features You Might Not Have Heard Of"
description: "Angular is a massive framework with lots of hidden goodies. Explore some lesser known features that could make your life easier."
image: assets/images/angular-unknown-features.png
slug: angular-unknown-features
date:   2018-08-08
---

Angular is an enormous framework, and most of us ever use only a fraction of it. But hiding in plain site are many useful features that can make your life simpler. In this article I am going to cover 12 useful Angular features that you may have not heard of.

One of the most easily forgotten, yet incredibly helpful parts of Angular are the pipes it provides, so we will start by discussing a few of the useful pipes!

###### 1\. KeyValuePipe

This was one of the key features of the Angular 6.1 release. Previously the `*ngFor` directive only allowed you to iterate over an Array or an Iterable construct. This was problematic when wanting to loop through properties in an object or items in a Map.

This is where the `KeyValuePipe` can help us out! We simply pipe the object we want to iterate over and `ngFor` takes care of the rest. For example:

{% highlight typescript %}
{% raw %}
@Component({
    selector: 'app-list',
    template: `
    <ul>
        <li *ngFor="let word of words | keyvalue">
            {{ word.key }}: {{ word.value }}
        </li>
    </ul>
    `
})
export class AppListComponent {
    words = new Map([[1, 'Angular'], [2, 'Guru']]);
}
{% endraw %}
{% endhighlight %}

KeyValuePipe Documentation: [https://angular.io/api/common/KeyValuePipe](https://angular.io/api/common/KeyValuePipe)

###### 2\. Slice Pipe

Rendering a list of items and manipulating large amounts of DOM nodes can be an expensive task, so there are occasions when we may want to reduce the number of items displayed. We could do this a number of ways, but Angular has a really simple `SlicePipe` that allows us to do this as part of our `ngFor` expression.

For example, lets say we want to display only the first 10 items of an array, we could do this using the `SlicePipe` like so:

{% highlight html %}
{% raw %}
<ul>
    <li *ngFor="let item of items | slice:0:10">{{ item }}</li>
</ul>
{% endraw %}
{% endhighlight %}

SlidePipe Documentation: [https://angular.io/api/common/SlicePipe](https://angular.io/api/common/SlicePipe)

###### 3\. Decimal Pipe

This is a very simple pipe is designed for formatting numbers. It can be very useful to limit the number of digits to be displayed after the decimal point, however it can also be used if you would simply like to have you number presented in a more readable fashion, with commas between every thousand, so 1000 would become 1,000.

One additional feature that is related to this pipe, is the `formatNumber`function that is part of `@angular/common` that allows you to apply the same formatting programmatically rather than having to do this in your view with a pipe.

{% highlight html %}
{% raw %}
<p>Your number is: {{ value | number }}</p>
{% endraw %}
{% endhighlight %}

DeimalPipe Documentation: [https://angular.io/api/common/DecimalPipe](https://angular.io/api/common/DecimalPipe)

formatNumber Documentation: [https://angular.io/api/common/formatNumber](https://angular.io/api/common/formatNumber)

###### 4\. JSON Pipe

The JSON pipe is incredibly useful, especially when debugging. It allows you to display an object in string for within your view. This can often be an easier alternative to having breakpoints and debugger statements.

Simply add the JSON pipe to the object you wish to display:

{% highlight html %}
{% raw %}
<p>{{ myObj | json }}</p>
{% endraw %}
{% endhighlight %}

JsonPipe Documentation: [https://angular.io/api/common/JsonPipe](https://angular.io/api/common/JsonPipe)

###### 5\. Title and Meta Services

These are particularly useful when it comes to working with search & social platforms, such as Google, Twitter, Facebook etc.. These platforms look for `<meta>` tags on your page to describe the content, providing titles, descriptions, images and more.

For example, this blog would have a different title, description and image for each post. To ensure each post gets the correct information we can use the Meta service which allows us to dynamically add and update the `meta` tags on the page, eg:

{% highlight typescript %}
{% raw %}
export class AppComponent {
    constructor(meta: Meta) {
    // Twitter Markup
    this.meta.updateTag({ name: 'twitter:card', content: 'summary_large_image' });
    this.meta.updateTag({ name: 'twitter:site', content: '@TheAngularGuru' });
    this.meta.updateTag({ name: 'twitter:title', content: 'Article title' });
    this.meta.updateTag({ name: 'twitter:description', content: 'Article description' });
    this.meta.updateTag({ name: 'twitter:image', content: 'image.jpg' });
    }
}
{% endraw %}
{% endhighlight %}

Additionally we have a `Title` service, which as you might have guessed, updates the title displayed in the browser window. This is normally achieved by setting the value of the `<title>` tag in the document head, however we can't use standard Angular bindings eg: `<title>{{ myTitle }}</title>` because `<head>` is not inside an Angular component, so we must instead use the `Title` service. It is incredibly easy to use:

{% highlight typescript %}
{% raw %}
export class AppComponent {
    constructor(title: Title) {
        title.setTitle('Window Title here');
    }
}
{% endraw %}
{% endhighlight %}

To allow social networking site to detect these meta tags they need to be present when the page loads (without JavaScript). This can be achieved by using [Angular Universal](https://angular.io/guide/universal).

Meta Documentation: [https://angular.io/api/platform-browser/Meta](https://angular.io/api/platform-browser/Meta)

Title Documentation: [https://angular.io/api/platform-browser/Title](https://angular.io/api/platform-browser/Title)

###### 6\. ng-container

Have you ever wanted to put two structural directives on the same element only to find out that you can't...

Or perhaps you are inserting a template using `ngTemplateOutlet`only to notice that this doesn't insert the template as a child, but instead as a sibling.

It can be very frustrating, and having to add a random wrapping `<div>` element in to accomplish this is not ideal, polluting your clean DOM structure.

Luckily there is an `ng-container` element that you can use to avoid such issues. This element, which present when developing, will not actually produce any new element in the DOM and we can use it to solver both of these issues and more!

For example, to have two structural directive we can simply do this:

{% highlight html %}
{% raw %}
<ng-container *ngIf="condition">
    <div *ngFor="let item of items">{{ item }}</div>
</ng-container>
{% endraw %}
{% endhighlight %}

And to insert a template as a child:

{% highlight html %}
{% raw %}
<div>
    <ng-container [ngTemplateOutlet]="template"></ng-container>
</div>
{% endraw %}
{% endhighlight %}

These are just a few of the many, many uses this element has, I'm sure you will find more!

###### 7\. Attribute Decorator

We all know the `@Input` and `@Output` property decorators which are used for bindings and emitting events, however there is another, lesser known property decorator, the `@Attribute` decorator.

This decorator in some ways is similar to the `@Input` decorator, as it can be used to pass a value to the component.

The attribute binding is quite similar to the literal scope binding of AngularJS:

{% highlight typescript %}
{% raw %}
scope: {
    type: '@'
}
{% endraw %}
{% endhighlight %}

First lets start with the limitations of the decorator!

*   Any values are provided as a string
*   Values are static and won't update like bindings do
*   We can't use the `[attribute]` binding syntax

While there are limitations, in occasions where they do not matter it can bring some decent performance benefits as these properties are not bindings they do not have to be checked during change detection.

They can used used just like the `@Input` decorator:

{% highlight typescript %}
{% raw %}
constructor(@Attribute() type: string) {}
{% endraw %}
{% endhighlight %}

Attribute Directive Documentation: [https://angular.io/api/core/Attribute](https://angular.io/api/core/Attribute)

Article on Attribute Directive: [https://netbasal.com/getting-to-know-the-attribute-decorator-in-angular-4f7c9fb61243](https://netbasal.com/getting-to-know-the-attribute-decorator-in-angular-4f7c9fb61243)

###### 8\. Profile Change Detection

I'm sure you are all aware of the `enableProdMode` function that will be called in the `main.ts` file of any Angular CLI project, however there is also an `enableDebugMode`function that can be called. You may think, well if I'm not running in production mode then I must be running in debug mode, and that is not the case (at least in terms of what happens when this function is called).

By calling this function we get an additional tool on the `window.ng` object called `profiler`. The profiler has a function called `timeChangeDetection` which if called will print to the console information on how many change detection cycles occur and the time it takes for them to run.

This can be a great help when trying to profile apps with poor performance. To call the function add the following to the bootstrap code:

{% highlight typescript %}
{% raw %}
platformBrowserDynamic().bootstrapModule(AppModule).then(ref => {
    const applicationRef = ref.injector.get(ApplicationRef);
    const appComponent = applicationRef.components[0];
    enableDebugTools(appComponent);
});
{% endraw %}
{% endhighlight %}

To start the profiling run the following in the DevTools console:

{% highlight typescript %}
{% raw %}
ng.profiler.timeChangeDetection();
{% endraw %}
{% endhighlight %}

###### 9\. NgPlural

There are cases in many applications where you need to describe a number of items, and with language being the complicated thing it is, we often have to deal with different phrasing based on different quantities. For example, consider some different possibilities:

*   There are no items
*   There is 1 item
*   There are 2 items

The `ngPlural` directive is an easy, built in way to handle cases like these. The example above could be handled like so:

{% highlight html %}
{% raw %}
<p [ngPlural]="value">
    <ng-template ngPluralCase="=0">There are no items</ng-template>
    <ng-template ngPluralCase="=1">There is 1 item</ng-template>
    <ng-template ngPluralCase="other">There are {{ value }} items</ng-template>
</p>
{% endraw %}
{% endhighlight %}

NgPlural Documentation: [https://angular.io/api/common/NgPlural](https://angular.io/api/common/NgPlural)

NgPluralCase Documentation: [https://angular.io/api/common/NgPluralCase](https://angular.io/api/common/NgPluralCase)

###### 10\. ngPreserveWhitespaces and NgNonBindable

I have grouped these two together since a common time to use them is with code snippets or any kind of formatted content.

In Angular 5 there was a `preserveWhitespaces` option added to the `angularCompilerOptions` . If this was set to false this allowed the compiler to remove any whitespace that was not deemed as essential. This can have minor spacing effects, but by in large it allows us to shrink our bundle size.

However there are occasions when we want to keep whitespace intact. If we want the whole component to retain its whitespace then we can simply use the option in the component decorator:

{% highlight typescript %}
{% raw %}
@Component({
    selector: 'app',
    templateUrl: './app.component.html',
    preserveWhitespace: true
})
{% endraw %}
{% endhighlight %}

We may however want to only retain whitespace within a specific DOM element in which case we can use the `ngPreserveWhitespaces` directive eg:

{% highlight html %}
{% raw %}
<div ngPreserveWhitespaces>
    <!-- All whitespace here will be preserved -->
</div>
{% endraw %}
{% endhighlight %}

Additionally we may want to use {% raw %}`{{ }}`{% endraw %} in our document, but Angular interprets this a use of interpolation and will try to evaluate whatever is within it. There is an `ngNonBindable` directive we can add to the parent element to inform Angular to ignore any braces, eg:

{% highlight html %}
{% raw %}
<span ngNonBindable>{{ this will not be evaluated }}</span>
{% endraw %}
{% endhighlight %}

So there you have it! There is my list of 10 things you my not have known Angular had available to use. Hopefully you will find a use for some of them!