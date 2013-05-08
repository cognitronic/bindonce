Bindonce
========

High performance binding for AngularJs

## Usage
* download, clone or fork it or install it using [bower](http://twitter.github.com/bower/) `bower install angular-bindonce`
* Include the `bindonce.js` script provided by this component into your app.
* Add `'pasvaz.bindonce'` as a module dependency to your app: `angular.module('app', ['pasvaz.bindonce'])`

## Overview
AngularJs provides a great data binding system but if you abuse of it the page can run into some performance issues, it's known that more of 2000 watchers can lag the UI and that amount can be reached easily if you don't pay attention to the data-binding. Sometime you really need to bind your data using watchers, especially for SPA because the data are updated in real time, but often you can avoid it with some efforts, most of the data presented in your page, once rendered, are immutable so you shouldn't keep watching them for changes.

For instance, take a look to this snippet:
```html
<ul>
	<li ng-repeat="person in Persons">
		<a ng-href="#/people/{{person.id}}"><img ng-src="{{person.imageUrl}}"></a>
		<a ng-href="#/people/{{person.id}}"><span ng-bind="person.name"></span></a>
		<p ng-class="{'cycled':person.generated}" ng-bind-html-unsafe="person.description"></p>
	</li>
</ul>
```
Angular internally creates a `$watch` for each `ng-*` directive in order to keep the data up to date, so in this example just for displaying few info it creates 6 + 1 (*ngRepeatWatch*) watchers per `person`, even if the `person` is supposed to remain the same once shown. Iterate this amount for each person and you can have an idea about how easy is to reach 2000 watchers. Now if you need it, because those data could change while you show the page, or are bound to some models, it's ok. But most of the time they are static data that don't change once rendered. This is where **bindonce** can really help you.

This is how **bindonce** handles the above example:
```html
<ul>
	<li bindonce ng-repeat="person in Persons">
		<a bo-href="'#/people/' + person.id"><img bo-src="person.imageUrl"></a>
		<a bo-href="'#/people/' + person.id" bo-text="person.name"></a>
		<p bo-class="{'cycled':person.generated}" bo-html="person.description"></p>
	</li>
</ul>
```
Now this example uses **0 watches** per `person` and renders the same result as the above that uses ng-*. *(Angular still uses 1 watcher for ngRepeatWatch)*

## The smart approach
OK until here nothing completely new, with a bit of efforts you could create your own directive and render the `person` inside the `link` function, or you could use [watch fighters](https://github.com/abourget/abourget-angular) that has a similar approach, but there is still one problem that you have to face and **bindonce** already handles it: *the existence of the data when the directive renders the content*. Usually the directives, unless you use watchers or bind their attributes to the scope (still a watcher), render the content when they are loaded into the markup, if at that given time your data are not available the directive can't render it. Bindonce can wait until the data are ready before to render the content. 
Let's give a look to the follow snippet to better understand the concept:
```html
<span my-custom-set-text="Person.firstname"></span>
<span my-custom-set-text="Person.lastname"></span>
...
<script>
angular.module('testApp', [])
.directive('myCustomSetText', function () {
	return {
		link:function (scope, elem, attr, ctrl) {
			elem.text(scope.$eval(attr.myCustomSetText));
		}
	}
});
</script>
```
This basic directive works as expected, it renders the `Person` datas and it doesn't use any watcher. However, if `Person` is not yet available inside the $scope when the page is loaded (say we get `Person` via $http or via $resource), the directive is useless, `scope.$eval(attr.myCustomSetText)` renders just nothing and exit.

Here is how we can solve this issue with **bindonce**:
```html
<div bindonce="Person" bo-title="Person.title">
	<span bo-text="Person.firstname"></span>
	<span bo-text="Person.lastname"></span>
	<img bo-src="Person.picture" bo-alt="Person.title">
	<p bo-class="{'fancy':Person.isNice}" bo-html="Person.story"></p>
</div>
```
`bindonce="Person"` does the trick, any `bo-*` attribute belonging to `bindonce` waits until the parent `bindonce="{somedata}"' is validated before to render its content. Once the scope contains the value `Person` then every bo-* children get filled with the proper values. In order to accomplish this job, **bindonce** uses just one temporary watcher, no matters how many children need to be rendered, as soon as it gets `Person` the watcher is properly removed.

## License
MIT