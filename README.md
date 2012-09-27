# Angular Table

This is a "Proof of concept" datagrid.

## Data Source

The data is generated by the random_transactions.js script - it is different for each view of the page.

To "Torture test" it against larger collections adjust the $scope.MONTHS_OF_DATA value in controller.js

### Useful variables

Each "Expense type" in random_transactions.js has the following fields.

Expense types are collected in named hashes; the name of the hash key is the type that will be assigned to the expense

 * `name`: the string name that appears in the "Expense" column. Some names have wildcards ("*") that are placeholders for tokens.
 * `tokens`: an array of strings to replace the "*" in the name.
 * `min and max`: the range of an expense's amount. Set to the same value for fixed amounts.
 * `inc`: for amounts with fixed increments (as in ATM withdrawals in units of 20) set this value to control which amounts are returned.
 * `min_freq and max_freq`: how many expenses of this type will be injected into each month.


## Angular app and Controller

An angular application is bound within an ng-app tag. ng-apps can be namespaced, as is done here with the "table" app, or clumped in to one anonymous namespace.

Every angular app has at least one controller. Controllers are bound to the ng-controller attribute of a dom element and have no ability to (directly) affect content outside that dom element.

Controllers can have functionality __injected__ in one of two ways:
 1. __named injection__ of general angular __services__ (such as the $resource component, useful for REST data IO) into the __controller__.
 2. __"functional injection"__ by (for example) adding named filters through repeated calls of the .filter() method of the __module__ itself.

### Controller $scope

Inside the body of the controller function is the $scope variable. $scope is essentially "this" for the controller. When you refer to a variable in the template, say {{article_title}} this maps to $scope.article_title.

## Templates and the refresh cycle

Every time you change a scope value, the template will re-render the value or any dependant calculations that it affects. Angular is pretty rigorous about this refreshing. If you want to take manual control of the refresh cycle, you can name a term to observe via $scope.$watch('foo', function(oldValue, newValue, $scope){ ... }) construct.

There are a few ways to connect a value to dom. All of these inject __expressions__ into your markup that can be scalars, functions, or calculations:

* __Template injection__: `<span>{{page_title}}</span>`. Can include limited calculations: `<span>{{ $index + 1}}</span>` or functions: '<span>{{last_name(user) }}</span>`
* __Attribute model injection__: `<input type="text" ng-model="page_title" />`, which will allow a user to edit data in real time with a form widget. (see also `ng-checked` for checkboxes)
* indirect mapping to particular attributes: `<span ng_class="item_class(data)">foo</span>`
* __repeat looping__: `ng-repeat` will express elements of an array through repeated tags, as in `<li ng-repeat="page in pages">{{page}}</li>`.
* __template insertion__: this is the "Backbone way" of slapping generated markup from a separate template into a dom element. This markup can include Angular markup and so, as such, can have dynamic attributes. A classic use case for this is to generate an edit form which is bound to a particular record, when the user clicks a button.

## Filtering and Ordering array data

Angular defines some "canned" ways to filter data. For instance you can search for a blanket term anywhere in a collection by declaring `<label>Filter: <input ng_model="searchterm"></label><tr ng-repeat="item in data | filter: searchterm"><td>{{item.name}}...</tr>`

You can filter for specific fields by using more granular search terms: `<label>Filter: <input ng_model="searchterm.name"></label><tr ng-repeat="item in data | filter: searchterm"><td>{{item.name}}...</tr>`

You can also pipe to an ordering term, as is shown in the template in this project. Note you really don't even need to write any code to accomplish most of these affects.

Filters are essentially functions and can be defined manually to alter scalar data, as with the `currency` filter in this example project. Any number of filters can be chained.

This is how I achieved pagination within this example with both the native Angular `limitTo` filter and a custom `startFrom` filter (essentially a slice wrapper).

## Interaction with rest services

You can bind directly to a REST endpoint with Angular. The process is very similar to that in Backbone and requires a `$resource` injection. Note that the `$resource` component is in a separate javaScript file from Angular core.

```language="javaScript"

angular.module('messages', ['messagesServices']);

angular.module('messagesServices', ['ngResource']).factory('Messages',
    function ($resource) {
        return $resource('/noogle/find_result/:size/:start?format=json',
            {size:'@size', start:'@start'}, {
                query:{method:'POST', fr:0}
            });
    });

function MessagesCtrl($scope, $filter, $compile, Messages) {

...


    $scope.find = function () {
        $('#wait').modal({show: true})

        Messages.query({start:'' + $scope.start, size:$scope.size, search: $scope.search, format:'json'},
            function (result) {
                $('#wait').modal('hide')
                $scope.messages = result.hits ? _.map(result.hits.hits, function (hit) {
                    var out = hit._source;
                    out.score = hit.score;
                    if (out.time){
                        out.time_date = new Date(out.time);
                    }
                    return out;
                }) : []
            }
        )
    }

    if ($scope.search){
        $scope.find();
    }

```

In the above example, you query for results from a remote dataset of messages by putting your search term in a local variable named `search` which is passed in as REST JSON in the body of a POST. the pagination is managed with the size and start properties of the URL itself.

By assigning the result of the callback to $scope.message, the template is updated when the query returns, after a little underscore-assisted filtering.

# Further Learning

The extensive [Angular Tutorial](http://docs.angularjs.org/tutorial) covers a lot of ground.

My [Noogle Search Engine](http://noogle.io/noogle/find?search=foo) also has a lot of interesting code for data parsing.