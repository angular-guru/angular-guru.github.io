---
layout: post
title:  "15 Angular Performance Tips & Tricks"
description: "Learn some useful tips and tricks to make your Angular application blazing fast!"
image: assets/images/angular-performance.jpg
slug: angular-performance-tips
date:   2018-08-08
---

Angular is a fantastic framework, packed with useful goodies to make developing an application much simpler. However Angular is not a small framework, and if certain optimizations are not made you can end up negatively impacting performance and your user experience.

For example, a newly created Angular CLI project built in development mode will produce JavaScript totaling 3mb in size, for essentially a hello world application! So it is essential to follow some best practices to make sure page load times, and general application performance remains fast!

#### 1\. Production Builds

Let’s start simple! When building your application to deploy it we want to ensure we do a production build. This will perform lots of optimizations as part of the build which are not included in a development build.

###### AOT Compilation

When running a production build, Angular using JIT (just in time) compilation, which essentially means, Angular compiles your views in the browser at runtime. This has two downsides. First, the compilation process must run before your application can be used, and this can increase the time it takes for your site to load. Secondly, we have to ship the Angular compiler with your application, and it is not a small module!

By taking advantage of AOT (ahead of time) compilation, we move this step to build time so we do it once when building our application, and only ship the compiled templates. We can now remove the Angular compiler from our bundle (reducing our bundle size by ~1mb) and allows us to skip the compilation step making our pages load much quicker!

###### Minification

Code minification is the process tools like UglifyJS perform to optimize the code we have written. It performs many optimizations, for example, removing whitespace, renaming properties, dead code elimination and much much more.

When developing having well named variables make development much easier, but when shipping our applications we don’t need these names to be so helpful, so while `averageUserAge` might be useful when developing this could be renamed to `a1` reducing the amount of code needed to be shipped.

It can also detect code paths that will never be executed, for example an if statement where the condition can never be satisfied:

{% highlight typescript %}
{% raw %}
if (false) {
    console.log('This would never be executed');
}
{% endraw %}
{% endhighlight %}

We can safely remove this code without worrying about breaking our application.

We can also minify our stylesheets as well. We generally aren’t able to make quite as significant savings as we can with JavaScript code, but it can remove unneeded whitespace and shorten color values eg: `#ffffff` to `#fff` along with some other tweaks as well.

This process can reduce our bundle size often by megabytes.

###### Build Optimizer

This is a tool created by the Angular team to identify some additional code that can be removed at build time.

For example, it can mark certain functions as “pure” indicating to UglifyJS that these functions can be removed without side effects.

If you have ever looked at compiled TypeScript code that uses decorators you may notice that it generates quite a lot of code to use them. The build optimizer can optimize this code for Angular decorators and reduce the code required quite significantly. It performs some other optimizations which can be found here: [https://github.com/angular/angular-cli/tree/master/packages/angular\_devkit/build\_optimizer](https://github.com/angular/angular-cli/tree/master/packages/angular_devkit/build_optimizer)

If you are using the CLI you can ensure the `buildOptimizer` flag is set to true in your `angular.json` file.

###### Running a Production Build

You can perform a production build that automatically performs all the optimizations mentioned above by adding the `--prod` flag when running an `ng build`.

#### 2\. Lazy Loading Modules

Most applications will have more than one pages, for example, you may have a home page, a login page and a profile page. By default, when your application starts, the browser will load all the code required for all of these pages even if the user never visits them. Luckily, Angular provides us with an easy way to only load pages when the user wants to navigate to them and this comes built in to the router.

As a prerequisite to this, each page you wish to lazy load must have its own NgModule that imports the `RouterModule` and provides its own routes using the `forChild` function.

Note, it is best not to lazy load the default route, as this is the first page most users will land on, and by lazy loading it the user will have to wait on an additional request.

Before Lazy Loading:

{% highlight typescript %}
{% raw %}
const routes: Routes = [
    {
        path: '',
        component: HomeComponent
    },
    {
        path: 'login',
        component: LoginComponent
    }
];
{% endraw %}
{% endhighlight %}

After Lazy Loading:

{% highlight typescript %}
{% raw %}
const routes: Routes = [
    {
        path: '',
        component: HomeComponent
    },
    {
        path: 'login',
        loadChildren: './login/login.module#LoginModule' 
    }
]; 
{% endraw %}
{% endhighlight %}

Now our application will only load the code for these pages when it is essential giving us a quick and easy improvement to our application.

#### 3\. Enable Production Mode

This is a very simple one, and one that your application is likely already doing if you are using the CLI, but it’s extremely simple to do, and beneficial to ensure you are doing it.

Angular, by default runs in debug mode, which essentially adds in some assertion checks and more importantly (at least for performance) it runs ChangeDetection twice each time to ensure there are no unexpected changes to values. You have likely come across the error `expression has changed after it was checked`, which occurs due to this second change detection pass.

It is very simple to enable production mode, and is usually done like this:

{% highlight typescript %}
{% raw %}
if (environment.production) {
    enableProdMode();
}
{% endraw %}
{% endhighlight %}

#### 4\. OnPush Change Detection

When an event occurs (eg. dom event, timeout, interval, http request etc…) Angular runs change detection so see if there are any values that have changed that require the view to be updated. This process is very fast, especially compared to AngularJS. This is largely down to the unidirectional data flow now used in Angular. However, even with the great performance improvements, as your application grows this can become slower.

By default, Angular will check every component that may have been affected to see if there have been changes, but we can be smarter, and tell Angular to only run change detection when an Input changes or when we manually trigger it. This will allow us to skip change detection in this component in most cases giving us a speed boost.

We can do this by specifying the change detection strategy in the component decorator:

{% highlight typescript %}
{% raw %}
@Component({
    selector: 'my-progress-bar',
    templateUrl: './progress-bar.html',
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class ProgressBar {
    @Input() value: number = 0;
}
{% endraw %}
{% endhighlight %}

OnPush change detection is easily used when using a reactive state management solution like NGRX - we will cover more about this later.

#### 5\. Preserve Whitespaces

This is another dead easy optimization that can reduce your bundle size by a small amount, but every little bit helps! In Angular 5, they introduced a new feature called `preserveWhitespaces` which allows you to tell the compiler to remove any whitespaces between elements. By default the compiler includes these as it can have a small effect on spacing.

In your applications `tsconfig.json` you can add the following to tell the compiler to remove whitespaces in all templates by default:

{% highlight json %}
{% raw %}
"angularCompilerOptions": {
    "preserveWhitespaces": false
}
{% endraw %}
{% endhighlight %}

We can also do this on a per-component basis and we can access this property in the component decorator, eg:

{% highlight typescript %}
{% raw %}
@Component({
    selector: 'my-progress-bar',
    templateUrl: 'progress-bar.component.html',
    preserveWhitespaces: false
})
{% endraw %}
{% endhighlight %}

Finally, we may want to preserve whitespaces for a particular element, which we can do by using a directive:

{% highlight html %}
{% raw %}
<div ngPreserveWhitespaces>

    <p>Any Whitespaces in this div will be retained</p>

</div>
{% endraw %}
{% endhighlight %}

#### 6\. Avoid Function Calls in Views

So this is a simple best practice to follow when writing your components. You should avoid calling functions directly from your view. There may be some cases where this is unavoidable, but for most cases this can be avoided.

Let’s say we have the following view:

{% highlight html %}
{% raw %}
<h1>{{ header }}</h1>
<h5>{{ getSubtitle() }}</h5>
{% endraw %}
{% endhighlight %}

Below is a simple pseudo code example of roughly what happens when change detection is run:

{% highlight typescript %}
{% raw %}
if (header !== previousHeader) {
    // do update
}

if (getSubtitle() !== previousSubtitle) {
    // do update
}
{% endraw %}
{% endhighlight %}

As you can see here every time change detection runs, we cannot simply check whether the subtitle has changed. We instead have to call the function before we can perform the check. Function calls have overhead and if you have this kind of code throughout your application this will start to have a noticeable effect!

In most cases, values are updated as part of an event or when an http request returns, so use these to store the latest value in a variable instead, eg:

{% highlight typescript %}
{% raw %}
export class AppComponent {
    header: string = 'App Header';
    subtitle: string;

    constructor(http: HttpClient) {
        http.get('https://abc.xyz/api/subtitle', (result) => this.subtitle = result);
    }
}
{% endraw %}
{% endhighlight %}

#### 7\. Avoid Getters in View

This is essentially an addendum to the previous point. It is quite common to use getters/setters when writing component classes. They can be incredible useful, but while you interact with them like you would any other variable, it is important to remember that they are in fact functions even though you don’t use them like one.

As a result, using a getter in the view is the same as calling a function as shown above. Avoid these if possible, and where possible use a separate variable to store the value accessed by the view, eg:

{% highlight typescript %}
{% raw %}
export class AppComponent {

    set name(value: string) {
        // do stuff here when the name value changes
        this.viewName = value;
    }

    // rather than have a getter that is accessed in the view, store the latest value here
    viewName: string;
}
{% endraw %}
{% endhighlight %}

#### 8\. Pure Pipes

As mentioned before, sometime you need to call a function in the view, but in many cases we can use a Pipe instead (more importantly a pure pipe). What is the difference between a pipe and a pure pipe? A pure pipe is a pipe that given the same input value will always return the same output, and this is a very important differentiation when it comes to optimization.

Lets take a look at simple pipe that takes in some text and makes it lowercase:

{% highlight typescript %}
{% raw %}
@Pipe({
    name: 'lowercase',
    pure: false
})
export class LowerCasePipe implements PipeTransform {
    transform(value: string): string {
        return value.toLowerCase();
    }
}
{% endraw %}
{% endhighlight %}

This can be used in our view like this:

{% highlight html %}
{% raw %}
<h1>{{ name | lowercase }}</h1>
{% endraw %}
{% endhighlight %}

Currently the pipe is marked as impure, which means, every time change detection is run this transform function will be called to see if the returned value has changed. As we have already discussed this is not really a good idea.

But lets look at the code for this pipe. Notice, the transform function only uses the `value` variable that is passed into it. It does not require any other external information and doesn’t relies on changing information such as the date or a random number. So we can know for sure if I pass in the string `AngularGuru` this pipe will always return `angularguru`.

We can now mark this pipe as pure (note pipes are pure by default), and Angular now knows, if the value passed into the pipe does not change then we do not need to called the function.

So consider making your own pure pipes when you need to call a function from the view!

#### 9\. The Angular Zone

In AngularJS we often run into times when our view didn’t update when it was supposed to because some asynchronous function updated data and AngularJS wasn’t aware of it. As a result we had to use the AngularJS alternatives for things like timeouts and intervals. This was fine for the most part but still commonly led to times where we had to manually trigger a digest (update the view).

As part of Angular 2 the team introduced a library called Zone.js. This library provided an execution context for our application to run in. While this sounds confusing, it essentially means they patch all asynchronous browser functions, so when an asynchronous function runs they know to perform change detection automatically, and for the most part, it works so much better than the AngularJS solution.

However, there are times when we are running some function that we know isn’t going to have an effect on data so we don’t need Angular to be aware of it. A few instances of this could be repeatedly called `requestAnimationFrame` to redraw the contents of a canvas, or using `setInterval` to print out to the console. Unless you otherwise specify, Angular will always run change detection when these events occur which can have pretty severe performance impacts on your application. Luckily, it is really easy to avoid!

Lets say we have an interval that prints the time out to the console every 100ms:

{% highlight typescript %}
{% raw %}
export class AppComponent {
    constructor() {
        setInterval(() => console.log(new Date().getTime()), 100);
    }
}
{% endraw %}
{% endhighlight %}

As mentioned this would run change detection every 100ms unnecessarily. To avoid this, we can inject `NgZone` and run this interval outside of the Angular zone.

{% highlight typescript %}
{% raw %}
export class AppComponent {
    constructor(ngZone: NgZone) {
        ngZone.runOutsideAngular(() => {
            setInterval(() => console.log(new Date().getTime()), 100);  
        });
    }
}
{% endraw %}
{% endhighlight %}

And that’s it! Problem solved!

#### 10\. Check your ngDoCheck

Angular provides us with a range of lifecycle hooks we can take advantage of in our components. One such hook is `ngDoCheck` which is called each time whenever change detection is run.

As a result, this function is going to get called a lot! If you are doing anything in this function that is in anyway computationally intensive or slow, you are going to experience slow down and there is likely a better place to be doing it.

#### 11\. Async Pipe

This is one of my favorite little Angular utilities. It essentially allows us to use RxJS observables directly in our view. Observables are used quite heavily in parts of Angular and once you get over the initial learning curve they provide a great reactive programming experience!

But there is more!

You need to be careful when using observables as it is very easy to subscribe to one, and then forget to unsubscribe, which can cause memory leaks in your application, which over time can cause slowdown. By using the Async pipe Angular automatically handles all the cleanup for you.

Probably the best thing about the Async pipe is that is allows us to make great use of OnPush change detection. Because we can subscribe to an observable, we can know exactly when the value has changed. There is no guess work involved. Normally Angular waits for an event and then checks to see if there have been any changes, but with observables we can flip this, as we know when a change has occurred we can inform Angular and intelligently update the view only when we are certain it needs to - and the Async pipe also does this for us automatically.

This works great with state management libraries such as NGRX which if fully utilized we can essentially make our entire application use OnPush change detection as all our state is handled through observables.

Using the Async pipe is as simple as:

{% highlight html %}
{% raw %}
<h1>{{ nameObservable | async }}</h1>
{% endraw %}
{% endhighlight %}

#### 12\. Unsubscribe

As previously mentioned, Angular uses observables quite a lot, and if you make any HTTP requests or are listening to router events, you will too.

Observables are great, but you need to ensure once you are finished with them that you unsubscribe, otherwise memory leaks can occur and this can cause performance issues.

Unsubscribing is easy, you store the subscription, and then use the `ngOnDestroy` lifecycle hook to unsubscribe, eg:

{% highlight typescript %}
{% raw %}
export class AppComponent implements OnDestroy {
    private _subscription: Subscription;

    constructor(router: Router) {
        this._subscription = router.events.subscribe(event => {
            // do stuff here
        });
    }

    ngOnDestroy(): void {
        this._subscription.unsubscribe();
    }
}
{% endraw %}
{% endhighlight %}

If you have multiple subscriptions there is an `add` function available on the subscription object which you can store all subscriptions in one object, however an alternative approach is to have an observable that will emit when the component is destroyed and it will automatically unsubscribe all others. For example:

{% highlight typescript %}
{% raw %}
export class AppComponent implements OnDestroy {

    private _onDestroy = new Subject<void>();

    constructor(router: Router) {
        router.events.pipe(takeUntil(this._onDestroy)).subscribe(event => {
            // do stuff here
        });
    }

    ngOnDestroy(): void {
        this._onDestroy.next();
        this._onDestroy.complete();
    }
}
{% endraw %}
{% endhighlight %}

#### 13\. Track By Function

Manipulating the DOM is an expensive task, and this can be very evident when it comes to rendering long lists of items, usually achieved by using the `*ngFor` directive.

By default, `ngFor` performs a simple equality check to see if items have changed. This is fine when it is a list of simple primitives such as numbers or strings, but can become a little bit more complicated when it comes to lists of objects.

As mentioned, it performs a simple equality check, which simply checks if the two objects are the same by reference, not by the properties within them.

It is common when using any Redux style architecture to enforce immutability, in other words any time the list of objects changes, each object within it will be a new object, and have a different reference even though the contents of it may be the same. As a result when `ngFor` performs it’s equality check, it will think the entire list contents have changed causing a complete re-render. Not exactly ideal for making your application performant.

The `ngFor` directive does however give us a simple solution in the form of a `trackBy` function. This is a function that we can provide to determine if the object is the same or not. For example, each object may have a unique id, which we can use to see if the item has changed. We can use this feature like so:

{% highlight html %}
{% raw %}
<ul>
    <li *ngFor="let document of documents; trackBy: trackByFn">{{ document.name }}</li>
</ul>
{% endraw %}
{% endhighlight %}

And that’s it! The `ngFor` directive can now perform efficient updates.

#### 14\. Profiling

There are many things that I can list to improve performance and even if your application followed everything listed here you may still have performance issues. That is simply because each application is different, it will use different third party libraries and be architected differently. And this is where profiling comes in!

The developer tools for all modern browser come equipped with performance profiling tools to help identify code that is running slowly, which is great to help figure out how you can improve it further.

There are a few other tools you can use to help improve performance, first the Webpack Bundle Analyzer. This tool allows you to visually explore your bundle. It can let you see what modules or libraries are the largest, but more importantly it can help identify items that should not have been included in the bundle. For example if you are using an older version of RxJS, accidentally importing directly from `rxjs` would have included the whole library in your bundle. Tools like this can help spot this kind of mistake and allow you to easily rectify it.

Lighthouse testing is another great way to see how you application performs on a range of devices. It is a tool now built in to the Chrome Dev Tools. It will profile many aspects of your application, such a load performance, accessibility, PWA support, SEO optimization and Best Practices. It can also give you a good indication about how well your site will perform in regards to Google rankings as many of these criteria it tests for will affect the site ranking.

The Angular CLI also comes with a useful tool called Budgets. You can configure these within the `angular.json` file. They allow you to impose file size limits on your scripts and stylesheets to ensure their file size doesn’t grow beyond acceptable limits. You can specify when it should warn you and when it should throw an error. This is very useful when it comes to accidental imports, as mentioned before, if you use budgets and then accidentally imported the entire RxJS library, budgets would immediately warn you that your bundle size has increased dramatically, prevent this from going unnoticed for an extended period of time.

#### 15\. Notable Mentions

Lastly I will mention some additional things you can do to increase performance. They are slightly less related to the code of your application and more so the configuration and server setup.

###### Compression

This is an obvious choice but often overlooked. Most browsers nowadays support gzipped resources. This allows your server to compress the website resources and send them to the browser. This can decrease the amount of data transmitted by ~75%. There are a vast range of different servers out there so I will not cover examples of how to configure it, but it is usually simply a matter of turning it on in a configuration file or adding in some middleware in the case of `express`.

###### Server Side Rendering

Angular provides several options when it comes to server side rendering. We have the full Angular Universal option which will render your website on the server and send it to the browser when the Angular application will take over and make it interactive. This is great for SEO and essentially required when it comes to adding social media metadata. It is fairly easy to setup, in fact this site uses Angular Universal, however it does require you to have a Node based server. If you are interested in this take a look at [@ng-toolkit](https://github.com/maciejtreder/ng-toolkit) which provides schematics to set it all up for you.

There are also additional options like AppShell which will pre-render the default route into the index.html for faster start up times and [Prerender](https://prerender.io/) which is another alternative.

###### Progressive Web Application

A progressive web application, utilizes local browser caching to allow your application to work similar to a native mobile application, providing most features even when offline. It will also be able to serve resources from a local cache reducing the number of network requests needed.

There are [schematics for the Angular CLI](https://angular.io/guide/service-worker-getting-started) to make the setup process incredibly easy.

###### Web Workers

Lastly, I will briefly mention this option. This is not widely used and there are some limitations with this option but if your application does some computationally intensive tasks that make the UI unresponsive this can be a great solution. Angular provides support to run the framework in a background thread, leaving the user experience fluid and responsive regardless of what processing you are doing.

#### Conclusion

As you can tell performance is not a simple subject. There are many things to consider and this list cover a few of the most common. Hopefully it covers a few of the most common and helps speed up your application.