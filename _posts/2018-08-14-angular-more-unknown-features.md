---
layout: post
title:  "10 More Useful Angular Features You Might Not Have Heard Of"
description: "Discover 10 more lesser known Angular features to boost your in depth knowledge of the framework!"
image: assets/images/angular-more-unknown-features.jpg
slug: angular-more-unknown-features
date:   2018-08-14
---

Here I will outline 10 more useful Angular features that you may not know about! If you haven't read our first post on the topic, [check it out here!](https://angular-guru.com/blog/angular-unknown-features)

### 1\. App Initializers

Quite often you want to run some code as soon as your application starts, for example to load some settings that will determine how parts of your application will appear. In this case let's say that the settings are loaded asynchronously, which can be problematic as the application continues it bootstrapping process even though you may not yet have all the settings you need.

The common solutions that spring to mind are perhaps loading the settings from the AppModule constructor, but that does not solve the issue of our app continuing to bootstrap. We could manually bootstrap the application, but this is getting a bit messy. We have some other alternatives like route guards for example, however this doesn't help if we use some settings in the AppComponent.

Luckily Angular provides us with a simple solution. We have access to an `APP_INITIALIZER` token that we can use to add functions that are called as part of the app initialization process. These functions can return a promise, so we can use them for asynchronous events as well.

{% highlight typescript %}
{% raw %}
export function onInit(settingsService: SettingsService) {
    return () => settingsService.getSettings();
}

@NgModule({
    providers: [
        SettingsService,
        { provide: APP_INITIALIZER, useFactory: onInit, deps: [SettingsService], multi: true }
    ]
})
export class AppModule { }
{% endraw %}
{% endhighlight %}

<a href="https://stackblitz.com/edit/angular-qrcnnl" target="_blank">Try on StackBlitz</a>

### 2\. Gesture Recognition

Nowadays we spent more and more time using devices other than a laptop or desktop. Last week, this site had more visitors using mobile devices than PCs, so mobile friendly sites are becoming more and more important. One important aspect of mobile sites is how the user interacts with the content. Instead of clicking or using a keyboard to navigate, they instead use touches and gestures.

Angular comes pre-built with all the browser event listeners, which largely are aimed around PC users and traditional mouse and keyboard events, along with some basic touch recognition events. There are however many touch gestures that could be utilized for better user experiences such as panning, zooming, swiping and rotating to name a few. Angular doesn't come with these event listeners as standard, however it comes with great support for HammerJS, a library designed for handling such events.

If we include this third party library, Angular will provide us with the output events to use these gestures as we would a click or key press, eg:

{% highlight html %}
{% raw %}
<div (tap)="onTap($event)"
     (press)="onPress($event)"
     (pressup)="onPressUp($event)"
     (pinch)="onPinch($event)"
     (pinchstart)="onPinchStart($event)"
     (pinchmove)="onPinchMove($event)"
     (pinchend)="onPinchEnd($event)"
     (pinchcancel)="onPinchCancel($event)"
     (pinchin)="onPinchIn($event)"
     (pinchout)="onPinchOut($event)"
     (swipe)="onSwipe($event)"
     (swipeleft)="onSwipeLeft($event)"
     (swiperight)="onSwipeRight($event)"
     (swipeup)="onSwipeUp($event)"
     (swipedown)="onSwipeDown($event)"
     (rotate)="onRotate($event)"
     (rotatestart)="onRotateStart($event)"
     (rotatemove)="onRotateMove($event)"
     (rotateend)="onRotateEnd($event)"
     (rotatecancel)="onRotateCancel($event)">
</div>
{% endraw %}
{% endhighlight %}

<a href="https://stackblitz.com/edit/pan-gesture" target="_blank">Try on StackBlitz</a>

Example courtesy of angularindepth.com

### 3\. Template Type Checking (and how to bypass it)

When Angular 2 was released, they heavily promoted the use of TypeScript for a better development experience and more importantly less bugs through type checking. This has been widely adopted and is now the standard for Angular applications, with plain old JavaScript rarely being used.

There are occasions in TypeScript where we perhaps don't have sufficient type definitions, for example, perhaps we are using a library that doesn't have them provided and we don't have the available time or resources to create them. TypeScript allows us to use a special type in this case called `any` which just tells the compiler to "keep quiet, I know what I'm are doing"...

As part of the AOT compilation process not only will type checking be performed on your TypeScript code, but will also type check your templates. This can be great for detecting errors, such as spelling mistakes or accessing properties that don't exist. As with TypeScript however, there can be times where you "know better", and are accessing some property that does exist even if the compiler doesn't know about it. Luckily we have an easy solution by using the `$any` type casting function, eg:

{% highlight html %}
{% raw %}
<p>{{ $any(user).name }}</p>
{% endraw %}
{% endhighlight %}

### 4\. Provider Scoping

Most of the time when creating a service within your application we either add it to the `providers` section of our `NgModule` eg:

{% highlight typescript %}
{% raw %}
@NgModule({
    providers: [ApiService]
})
{% endraw %}
{% endhighlight %}

Or, as we now should be doing in Angular 6 and above:

{% highlight typescript %}
{% raw %}
@Injectable({
    providedIn: 'root'
})
export class ApiService {}
{% endraw %}
{% endhighlight %}

Which will make our service tree shakable if it is not used.

However this creates a singleton, which means the service will be instantiated once and anywhere and the same reference will be provided to anything that gets it injected (Note, there is an exception when using services in lazy loaded modules).

Sometimes, a service can be a very useful way of communicating between parent and child components, for example we may have some components that are used to select items:

{% highlight html %}
{% raw %}
<app-check-list>
    <app-check-list-item value="Item One"></app-check-list-item>
    <app-check-list-item value="Item Two"></app-check-list-item>
    <app-check-list-item value="Item Three"></app-check-list-item>
</app-check-list>
{% endraw %}
{% endhighlight %}

We could use a simple service to allow the selection state to be changed by the check list item and known by the check list, eg:

{% highlight typescript %}
{% raw %}
@Injectable()
export class SelectionService {

    selection: string[] = [];

    select(item: string): void {
        this.selection = [...this.selection, item];
    }

    deselect(item: string): void {
        this.selection = this.selection.filter(_item => _item !== item);
    }
}
{% endraw %}
{% endhighlight %}

The `app-check-list` component could inject this to get the current selection state, and the `app-check-list-item` components could inject this to select and deselect themselves, keeping everything nicely in sync!

However... as mentioned, services are singletons. If we have more than one checklist we could have some big issues. How would we know which selected items belong to which list? I suppose we could add some additional property to identify which list the items belong to, but this is getting convoluted and removes the simplicity we had.

The solution is simple and Angular provides it right out of the box. We can scope services to components! The `@Component` decorator comes with two useful properties to achieve this:

{% highlight typescript %}
{% raw %}
@Component({
    selector: 'app-check-list',
    templateUrl: './check-list.component.html',
    providers: [
        SelectionService
    ]
})
{% endraw %}
{% endhighlight %}

The first is the `providers` property, which you can specify the service here and this means any time this component is initialized a new instance of the service will be created and any child element that injects the service will receive the one provided by this component.

Note, we also no longer need to add the service to the `NgModule` or state that it is `providedIn: 'root'`.

As mentioned, there is a second property on the `@Component` decorator that can be useful here. The property is called `viewProviders` which essentially does the same as the `providers` property, however this limits the scope even further by only exposing this service to components that are in the component template. Any child inserted using `<ng-content></ng-content>` which not have access to this service. So the example shown above of the check list would have to use `providers` as the `app-check-list-item` components are content and not part of the `app-check-list` template.

<a href="https://stackblitz.com/edit/angular-8vd8ma" target="_blank">Try on StackBlitz</a>


### 5\. Host, Self, SkipSelf & Optional Decorators

Angular comes with quite a few useful decorators, but we usually only use a couple of the on a day to day basis, namely `@NgModule`, `@Component`, `@Directive` and `@Injectable`. But Angular does have some additional decorators that allow us add additional constraints to dependency injection and how it resolves a dependency.

First it's important to have a basic knowledge of how Angular's Injector works. If we have a component that is looking to inject a service Angular will begin by checking the component itself to see if it registers any Injectables, like shown in the section above using the `providers` or `viewProviders` property on the `@Component` decorator. If no match is found, it will continue to check the parent component to see if it provides the injectable. This process continues up the injector tree until it finds what it is looking for. If no matching injectable can be found you will get an error stating that there was no provider found.

###### Self Decorator

The `@Self` decorator can be used to limit how far up the Injector tree we go to look for an Injectable. `@Self` states that we should check the current component for the injectable and look no further up the tree. Simple!

###### Host Decorator

The `@Host` decorator is similar in many respects to the `@Self` decorator. It also limits how far up the tree will be checked. It is also scoped to the component itself with a few additional edge cases:

*   If it is a directive that is requesting an injectable it will look on the component that the directive is applied on.
*   If the component is inserted using `<ng-content>` the component containing the `<ng-content>` element will also be checked.

###### SkipSelf Decorator

The `@SkipSelf` decorator is essentially the exact inverse of the `@Self` decorator. It will not check itself and will only look on parents to find the injectable.

###### Optional Decorator

The `@Optional` decorator can be very useful in cases where we may not always find the Injectable we are looking for, but we are prepared for such eventualities. This will prevent any errors being thrown if the injectable we are searching for isn't found, just be sure your code still works without the injectable.

### 6\. Http Interceptors

Http Interceptors are very helpful when it comes to making authenticated requests. When using an authentication method such as JSON Web Tokens, we need to send a header on any requests to allow the server to validate our identity and ensure we have access to the resource we are requesting.

It can be tedious to have to add the `Authorization` header to each `HttpClient` request we make, so Http Interceptors allow us to specify this once and have it done automatically for every request. We can simply create an injectable that implements the `HttpInterceptor` interface and add it to our list of providers, eg:

{% highlight typescript %}
{% raw %}
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
    intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
        const token = localStorage.get('token');

        request = request.clone({
            setHeaders: {
            Authorization: `Bearer ${token}`
            }
        });
        return next.handle(request);
    }
}
{% endraw %}
{% endhighlight %}

Then in our module:

{% highlight typescript %}
{% raw %}
@NgModule({
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: TokenInterceptor,
            multi: true
        }
    ]
})
export class AppModule {}
{% endraw %}
{% endhighlight %}

And that's it!

### 7\. Route Guards

While we are on the topic of user permissions and security, the router comes with a great feature called Guards, which allow us to check whether or not a user is allowed to access a specific page in your application and prevent them from navigating to it is necessary.

For example, let's say you have a profile page, where users can go to view and update their profile. This page should not be accessible to people who have not logged it. So first we would obviously not show the button or link that would take the user to the profile page, however that does not stop them from navigating to the URL directly, which is where router guards come in.

Now it is important to remember, that ABSOLUTELY NO front end security measures are enforceable and you need to ensure you provide the proper security on the server side. JavaScript code can easily be altered in the browser to allow access even with features such as route guards, so do not rely on them for security. Consider them a way to provide a better user experience if somewhere were to navigate to a URL they had previously accessed when logged in, but are now logged out.

A simple example of a route guard is:

{% highlight typescript %}
{% raw %}
@Injectable()
export class AuthGuard implements CanActivate {
    constructor(private authService: AuthService) {}

    canActivate(route: ActivatedRouteSnapshot, state: RouterStateSnapshot): boolean {
        return this.authService.isLoggedIn();
    }
}

Then to add this to a specific route:

{% highlight typescript %}
{% raw %}
const routes: Routes = [
    {
        path: 'profile',
        component: ProfileComponent,
        canActivate: [AuthGuard]
    }
];
{% endraw %}
{% endhighlight %}

Route Guards Documentation: [https://angular.io/guide/router#milestone-5-route-guards](https://angular.io/guide/router#milestone-5-route-guards)

### 8\. RxJS Subscriptions

While this is not directly Angular related, I thought I would include this anyway as RxJS is so commonly used throughout Angular.

If you have any knowledge about RxJS at all, you will know that you should always unsubscribe from an observable when we no longer need it. This can be simply achieved like so:

{% highlight typescript %}
{% raw %}
export class AppComponent implements OnDestroy {

    private _subscription: Subscription;

    constructor(router: Router) {
        this._subscription = router.events.subscribe(() => { /*...*/ });
    }

    ngOnDestroy(): void {
        this._subscription.unsubscribe();
    }
}
{% endraw %}
{% endhighlight %}

This works great, but all too often I see something like this when we have many observables:

{% highlight typescript %}
{% raw %}
export class AppComponent implements OnDestroy {

    private _mouseDownSubscription: Subscription;
    private _mouseUpSubscription: Subscription;
    private _mouseMoveSubscription: Subscription;

    constructor(@Inject(DOCUMENT) document: any) {
        this._mouseDownSubscription = fromEvent(document, 'mousedown')
            .subscribe(() => { /*...*/ });

        this._mouseUpSubscription = fromEvent(document, 'mouseup')
            .subscribe(() => { /*...*/ });

        this._mouseMoveSubscription = fromEvent(document, 'mousemove')
            .subscribe(() => { /*...*/ });
    }

    ngOnDestroy(): void {
        this._mouseDownSubscription.unsubscribe();
        this._mouseUpSubscription.unsubscribe();
        this._mouseMoveSubscription.unsubscribe();
    }
}
{% endraw %}
{% endhighlight %}

It's starting to get kind of messy and unmaintainable, but there are better ways.

###### Option 1

The subscription object has an `add` function, which allows us to store multiple subscriptions on the one object, eg:

{% highlight typescript %}
{% raw %}
export class AppComponent implements OnDestroy {

    private _subscription = new Subscription();

    constructor(@Inject(DOCUMENT) document: any) {
        this._subscription.add(fromEvent(document, 'mousedown')
            .subscribe(() => { /*...*/ }));

        this._subscription.add(fromEvent(document, 'mouseup')
            .subscribe(() => { /*...*/ }));

        this._subscription.add(fromEvent(document, 'mousemove')
            .subscribe(() => { /*...*/ }));
    }

    ngOnDestroy(): void {
        this._subscription.unsubscribe();
    }
}
{% endraw %}
{% endhighlight %}

###### Option 2

Personally I really like the following option, I find it scales great and works really well. In RxJS we have many operators that unsubscribe automatically for us. For example there are cases where we only actually care about the first value, or until a certain condition is met, and we have the `first` and `takeWhile` operators which will automatically unsubscribe for us in these cases. Another useful one is `takeUntil`, which we can use to unsubscribe all our observables when another emits, we can use this like so:

{% highlight typescript %}
{% raw %}
export class AppComponent implements OnDestroy {

    private _onDestroy = new Subject<void>();

    constructor(@Inject(DOCUMENT) document: any) {
        fromEvent(document, 'mousedown').pipe(takeUntil(this._onDestroy))
            .subscribe(() => { /*...*/ }));

        fromEvent(document, 'mouseup').pipe(takeUntil(this._onDestroy))
            .subscribe(() => { /*...*/ }));

        fromEvent(document, 'mousemove').pipe(takeUntil(this._onDestroy))
            .subscribe(() => { /*...*/ }));
    }

    ngOnDestroy(): void {
        this._onDestroy.next();
        this._onDestroy.complete();
    }
}
{% endraw %}
{% endhighlight %}

### 9\. Preload Lazy Modules

If you are familiar with Angular, you will likely have encountered lazy loading of modules. It comes built in to the router and makes code splitting incredibly easy, but most importantly, we can use it to speed up our app start time by only loading a page when the user actually navigates to it.

This can make significant performance and bundle size improvements, however we might be able to do slightly better. The one downside of lazy loading pages is that there is often a small delay the first time the user goes to a page while the scripts is loaded from the server and parsed by the browser.

Luckily Angular provides us with a way to get the best of both worlds, known as a preloading strategy. Essentially, this allows us to code split our application so we still only load the code for the page the user first goes to, but while the user is browsing the page, we can start loading some of these other pages in the background. This allows us to keep the initial page load time really fast, but take advantage of the browser "idle" time, so if they do happen to navigate to another page in the application, it is likely that we have already loaded it, giving them the instant page switching we have come to know any love in single page applications.

It is incredibly easy to set up, simply add the option to the `RouterModule` as part of the `forRoot` function:

{% highlight typescript %}
{% raw %}
imports: [
    RouterModule.forRoot(routes, { preloadingStrategy: PreloadAllModules })
],
{% endraw %}
{% endhighlight %}

It's also good to know you can easily create your own preloading strategy. For example you may have some pages that are more often visited than other so perhaps they should be preloaded with a higher priority.

PreloadAllModules Documentation: [https://angular.io/api/router/PreloadAllModules](https://angular.io/api/router/PreloadAllModules)

### 10\. Readonly Types for Immutability

This is another one that while not directly Angular specific can help greatly when using Angular features such as OnPush change detection and immutable state management such as NGRX.

Let's start by examining OnPush change detection. This is a way for Angular to optimize change detection by completely avoiding a component unless it knows an `@Input` has changed.

This is a great optimization which can significantly improve the performance of your application, however it does come with some limitations, most notably, how inputs are checked for changes.

To keep your application as fast as possible, Angular uses reference equality checks to determine if an Input has changed, in other words, something like this:

{% highlight typescript %}
{% raw %}
if (oldValue !== newValue) {
    // value has changes
}
{% endraw %}
{% endhighlight %}

This is fine when the input value is a primitive type such as a number, a boolean or a string, however we can start to experience some difficulties when using arrays or objects. JavaScript does not perform any kind of deep checking as part of the equality check it simply checks the reference, let's look at a simple example:

{% highlight typescript %}
{% raw %}
let obj1 = { name: 'Michael Scott' };
let obj2 = obj1;

// Change the name property in obj2
obj2.name = 'Dwight Schrute';

// Perform an equality check
if (obj1 === obj2) {
    // this will be true!!
}
{% endraw %}
{% endhighlight %}

So even though we changed a property on `obj2` the references are still the same, so as far as JavaScript is concerned these two objects are still equal!

The same goes for arrays, for example, pushing a new value to an array will also still return true in an equality test, even though there is a new item in the array!

In cases like this, updating an object property or adding an item to an array will NOT trigger change detection in a component with OnPush change detection enabled.

This is where the concept of immutable objects and arrays come in. Instead of changing a property in an object or pushing an item to an array, we create a new object or array, based off the original, but with the changes we require made. By making this change we change the object reference allowing our equality check to know a change had occurred.

With the new JavaScript [Spread operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax) we can make this really simple to do, eg:

{% highlight typescript %}
{% raw %}
let obj1 = { name: 'Michael Scott', company: 'Dunder Mifflin' };
let obj2 = { ...obj1, name: 'Dwight Schrute' };

if (obj1 === obj2) {
    // Objects are now not equal
}
{% endraw %}
{% endhighlight %}

We can also use the spread operator to add items to an array and create a new reference:

{% highlight typescript %}
{% raw %}
let employees = ['Pam Beasley'];
employees = [...employees, 'Jim Halpert'];
{% endraw %}
{% endhighlight %}

It can sometimes be difficult to ensure that we are always creating a new object or array, particularly with the vast number of array function available, some of which do and some don't. Take a look here to see the madness: [https://doesitmutate.xyz/](https://doesitmutate.xyz/).

This is where TypeScript can really come in useful. We can tell TypeScript to enforce immutability on an object or array, meaning any time we make a change to the without changing its reference we will get a build error. And it's super simple to do! Simply start using the `Readonly` and `ReadonlyArray` types!

{% highlight typescript %}
{% raw %}
// Before:
const employee: Employee = { name: 'Michael Scott' };

// This will compile fine
employee.name = 'Oscar Martinez'; // After:
const employee: Readonly<Employee> = { name: 'Michael Scott' };

// This will produce an error
employee.name = 'Kevin Malone';
{% endraw %}
{% endhighlight %}

We can also use readonly types for arrays, maps, and sets:

{% highlight typescript %}
{% raw %}
const employees: ReadonlyArray<Employee>;
const employees: ReadonlySet<Employee>;
const employees: ReadonlyMap<string, Employee>;
{% endraw %}
{% endhighlight %}

You don't need to include a library like Immutable.js to make your code immutable. Let TypeScript do it for you at build time!

<a href="https://stackblitz.com/edit/typescript-sebyyo" target="_blank">Try on StackBlitz</a>
