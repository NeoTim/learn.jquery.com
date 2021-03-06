<script>{
	"title": "jQuery Deferreds",
	"level": "advanced",
	"source": "http://msdn.microsoft.com/en-us/magazine/gg723713.aspx",
	"attribution": [
		"Julian Aubourg <j@ubourg.net>",
		"Addy Osmani <addyosmani@gmail.com>",
		"Andree Hansson <peolanha@gmail.com>"
	]
}</script>

## jQuery Deferreds

Deferreds were added as a part of a large rewrite of the Ajax module, led by Julian Aubourg following the CommonJS Promises/A design. Whilst 1.5 and above include deferred capabilities, former versions of jQuery had `jQuery.ajax()` accept callbacks that would be invoked upon completion or error of the request, but suffered from heavy coupling — the same principle that would drive developers using other languages or toolkits to opt for deferred execution.

In practice what jQuery's version provides you with are several enhancements to the way callbacks are managed, giving you significantly more flexible ways to provide callbacks that can be invoked whether the original callback dispatch has already fired or not. It is also worth noting that jQuery's Deferred object supports having multiple callbacks bound to the outcome of particular tasks (and not just one) where the task itself can either be synchronous or asynchronous.

At the heart of jQuery's implementation is `jQuery.Deferred` — a chainable constructor which is able to create new deferred objects that can check for the existence of a promise to establish whether the object can be observed. It can also invoke callback queues and pass on the success of synchronous and asynchronous functions. It's quite essential to note that the default state of any Deferred object is unresolved. Callbacks which may be added to it through `.then()` or `.fail()` are queued up and get executed later on in the process.

You are able to use Deferred objects in conjunction with the promise concept of when(), implemented in jQuery as `$.when()` to wait for all of the Deferred object's requests to complete executing (i.e. for all of the promises to be fulfilled). In technical terms, `$.when()` is effectively a way to execute callbacks based on any number of promises that represent asynchronous events.

An example of `$.when()` accepting multiple arguments can be seen below in conjunction with `.then()`:

```
function successFunc() {
	console.log( "success!" );
}

function failureFunc() {
	console.log( "failure!" );
}

$.when(
	$.ajax( "/main.php" ),
	$.ajax( "/modules.php" ),
	$.ajax( "/lists.php" )
).then( successFunc, failureFunc );
```

The `$.when()` implementation offered in jQuery is quite interesting as it not only interprets deferred objects, but when passed arguments that are not deferreds, it treats these as if they were resolved deferreds and executes any callbacks (`doneCallbacks`) right away. It is also worth noting that jQuery's deferred implementation, in addition to exposing `deferred.then()`, also supports the `deferred.done()` and `deferred.fail()` methods which can also be used to add callbacks to the deferred's queues.

We will now take a look at a code example that uses many deferred features. This very basic script begins by consuming (1) an external news feed and (2) a reactions feed for pulling in the latest comments via `$.get()` (which will return a promise-like object). When both requests are received, the `showAjaxedContent()` function is called. The `showAjaxedContent()` function returns a promise that is resolved when animating both containers has completed. When the `showAjaxedContent()` promise is resolved, `removeActiveClass()` is called. The `removeActiveClass()` returns a promise that is resolved inside a `setTimeout()` after 4 seconds have elapsed. Finally, after the `removeActiveClass()` promise is resolved, the last `then()` callback is called, provided no errors occurred along the way.

```
function getLatestNews() {
	return $.get( "latestNews.php", function( data ) {
		console.log( "news data received" );
		$( ".news" ).html( data );
	});
}

function getLatestReactions() {
	return $.get( "latestReactions.php", function( data ) {
		console.log( "reactions data received" );
		$( ".reactions" ).html( data );
	});
}

function showAjaxedContent() {
	// The .promise() is resolved *once*, after all animations complete
	return $( ".news, .reactions" ).slideDown( 500, function() {
		// Called once *for each element* when animation completes
		$(this).addClass( "active" );
	}).promise();
}

function removeActiveClass() {
	return $.Deferred(function( dfd ) {
		setTimeout(function () {
			$( ".news, .reactions" ).removeClass( "active" );
			dfd.resolve();
		}, 4000);
	}).promise();
}

$.when(
	getLatestNews(),
	getLatestReactions()
)
.then(showAjaxedContent)
.then(removeActiveClass)
.then(function() {
	console.log( "Requests succeeded and animations completed" );
}).fail(function() {
	console.log( "something went wrong!" );
});
```
