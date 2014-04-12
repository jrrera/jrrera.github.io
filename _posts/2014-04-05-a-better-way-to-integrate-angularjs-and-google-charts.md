---
layout: post
title: "A Better Way to Integrate AngularJS and Google Charts"
description: ""
category: "AngularJS"
tags: ['AngularJS', 'Google Visualization API']
---
{% include JB/setup %}

![AngularJS](/assets/img/AngularJS-large.png)

Last Updated: 4/12/2014


I'm fairly new to the world of web development (I [started coding](http://www.livingforimprovement.com/learning-to-code-fun-frustrating-fruitful/) a year and half ago), and one of my favorite discoveries thus far is AngularJS. 

As with many people new to Angular, the hardest concept to grok was that of directives. Whenever you'd like to manipulate the dom in some way, a directive is how you do it. Additionally, directives (along with services) are a great way to integrate third party libraries and APIs with Angular.

I've recently been working on a small [meditation timer app](https://github.com/jrrera/minimal-meditation) and was having some trouble integrating the Google Charts / Visualization API with AngularJS. I found a solid starting point with [Gavin Draper's article](http://gavindraper.com/2013/07/30/google-charts-in-angularjs/) on how he did it.

His code samples were fantastic for getting me up and running. But as I continued to work on the application, I found a few disadvantages with that implementation, and figured it never hurts to improve on the great content of others. Here are a few areas I wanted to work on:

### Bootstrapping Angular ###

{% highlight javascript %}
google.setOnLoadCallback(function () {  
    angular.bootstrap(document.body, ['my-app']);
});
google.load('visualization', '1', {packages: ['corechart']});
{% endhighlight %}

This is the code used to initialize the Angular application in Gavin's post. The given code here works just fine, but I noticed that Angular's bootstrap function only runs once the Google Loader fires off the callback function.

By requiring the Google Loader callback to fire before bootstrapping the Angular application, if the Google Loader ever falters, the app simply won't run. That felt like an unacceptable trade-off.

A better approach would be to wrap the Googe Loader in an Angular Service, which I'll explain how I did later in this post.

### Nested models ###

{% highlight javascript %}
googleChart.directive("googleChart",function(){  
    return{
        restrict : "A",
        link: function($scope, $elem, $attr){
            var dt = $scope[$attr.ngModel].dataTable;

            var options = {};
            if($scope[$attr.ngModel].title)
                options.title = $scope[$attr.ngModel].title;

            var googleChart = new google.visualization[$attr.googleChart]($elem[0]);
            googleChart.draw(dt,options)
        }
    }
});
{% endhighlight %}

The directive used in Gavin's article wasn't utilizing Angular's useful `$scope.$eval` for [reading attributes](http://stackoverflow.com/questions/15671471/angular-js-how-does-eval-work-and-why-is-it-different-from-vanilla-eval). Rather, it was passing the ngModel attribute directly to the `$scope` object. If you had a nested model – a best practice in many situations – such as `$scope.chartModel.dataset1`, the directive would break by trying to do something like this `var dt = $scope['chartModel.dataset1'].dataTable;`

As I mentioned, a better approach would be to use `$scope.$eval`, which will safely eval the attribute to give you access to the necessary model.

### Unit Testing ###

The only other concern I had was, by calling google.load in the global scope, unit testing became more difficult. My Karma / Jasmine setup was failing because the google namespace wasn't defined in the test environment when injecting my Angular app module. 

Again, the way to avoid this problem is to keep the google namespace wrapped in an Angular Service so that it can be mocked and/or ignored in my various unit tests.

### A Better Solution ###

Given the drawbacks listed above, here's how I ended up implementing Google Charts. I consider this to be a more 'Angular' way to do it, by relying more heavily on Angular services, watchers, and keeping the Google Loader in a nicely contained environment.

Let's start with the DOM and the controller:

__DOM__:

{% highlight html %}
<!DOCTYPE html>  
<html lang="en" ng-app="myApp">  
    <head>
         <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.2.15/angular.min.js"></script>
         <script src="https://www.google.com/jsapi" type="text/javascript"></script>
    </head>
    <body ng-controller="ChartCtrl">
        <div google-chart="ColumnChart" ng-model="dataModel.visual" trigger="activateChart"></div>
    </body>
</html>  
{% endhighlight %}

Note that `$scope.activateChart` is going to be the trigger to build the chart. 

__Controller__:

{% highlight javascript %}
var app = app || angular.module('myApp', []);

app.controller('ChartCtrl', function($scope, ChartService) {
    
    // activateChart flips to true once the Google 
    // Loader callback fires
    $scope.activateChart = false;

    // This is where my data model will be stored.
    // "visual" will contain the chart's datatable
    $scope.dataModel = {
        visual: {},
        metaData: {},
        data: {}
    };

    // First, we attempt to load the Visualization module 
    var loadGoogle = ChartService.loadGoogleVisualization();
    
    // If the Google Loader request was made with no errors, 
    // register a callback, and construct the chart data
    // model within the callback function
    if (loadGoogle) {

        google.setOnLoadCallback(function() {

            $scope.dataModel.visual.dataTable = new google.visualization.DataTable();

            // Set up the dataTable and columns
            var dataTable = $scope.dataModel.visual.dataTable;
            dataTable.addColumn("string","Date")
            dataTable.addColumn("number","Minutes")
            
            // Populate row data
            dataTable.addRow(["3/1/14",5]);
            dataTable.addRow(["3/2/14",13]);

            // Update the model to activate the chart on the DOM
            // Note the use of $scope.$apply since we're in the 
            // Google Loader callback.
            $scope.$apply(function(){
                $scope.activateChart = true;    
            });
        });  
    }
});
{% endhighlight %}

Next, let's look at the __Angular Service__. I chose to place google.load() in a try/catch block to guard against any errors that might spring up if any breaking changes are introduced in the future. 

There's also a quirk in the Google Loader worth noting: If you want to load an API _after_ the page renders, you need to add an arbitrary callback, otherwise the loader will use document.write(), which will overwrite all of the HTML on the page.

{% highlight javascript %}
app.factory('ChartService', function() {
    return {
        
        /**
         * Loads the visualization module from the Google Charts API 
         * if available
         * @returns {boolean} - Returns true is successful, or false 
         * if not available
         */
        loadGoogleVisualization: function() {
            
            // Using a try/catch block to guard against unanticipated 
            // errors when loading the visualization lib
            try {

                // Arbitrary callback required in google.load() to 
                // support loading after initial page rendering
                google.load('visualization', '1', {
                    'callback':'console.log(\'success\');', 
                    'packages':['corechart']
                });
               
                return true;
            
            } catch(e) {
                console.log('Could not load Google lib', e);
                return false;  
            }

        }
    };
});
{% endhighlight %}

And finally, let's take a look at the underlying directive.

__Directive__:

{% highlight javascript %}
app.directive("googleChart",function(){  
    return{
        restrict : "A",
        link: function($scope, $elem, $attr){
            var model;

            // Function to run when the trigger is activated
            var initChart = function() {

                // Run $eval on the $scope model passed 
                // as an HTML attribute
                model = $scope.$eval($attr.ngModel);
                
                // If the model is defined on the scope,
                // grab the dataTable that was set up
                // during the Google Loader callback
                // function, and draw the chart
                if (model) {
                    var dt = model.dataTable,
                        options = {},
                        chartType = $attr.googleChart;

                    if (model.title) {
                        options.title = model.title;
                    }
                    
                    var googleChart = new google.visualization[chartType]($elem[0]);
                    googleChart.draw(dt,options)
                }
            };

            // Watch the scope value placed on the trigger attribute
            // if it ever flips to true, activate the chart
            $scope.$watch($attr.trigger, function(val){
                if (val === true) {
                    initChart(); 
                }
            });
            
        }
    }
});
{% endhighlight %}

And there you have it! Google Visualization Charts right in your Angular application without any worries of Google Loader failure, unit testing difficulties, or data model restrictions. 

Granted, I'm still a beginner to AngularJS (I've only been writing Angular apps for 7-8 months now), so if you see any faults here, definitely point it out in the comments below! For starters, it probably would've made more sense to register the Google Loader callback in the service, rather than the controller. Comment if you agree!

-Jon