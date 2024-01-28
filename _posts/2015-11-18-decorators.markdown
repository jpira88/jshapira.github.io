---
layout: post
title:  "Using decorators to replace and tweak 3rd party services for your specific needs"
date:  2015-11-18 18:33:28 +0200
categories: jekyll update
author: Jacob Shapira
tags: ["FE", "JS/TS"]
---

* TOC
{:toc}


### Intro
Decorators allows us to enrich, or replace certain part of module we are not allowed (or shouldn’t) touch it’s source code. For example: 3rd party libraries, modules that we are not maintaining and the list can go on. 

### Real-life scenario
Let’s say we have file upload functionality in our application, and for that, we are using a great module we found ngmodules.org. For the sake of the example, let’s work with ng-file-upload.
So, we are extremely satisfied with this module, but then one bright day the requirements of the file upload changed. From now on, we need to upload those files to a remote machine, using special access credentials.

So what will we do? Should we replace the ng-file module? We can’t, we’re using it’s other nice features like drag and drop, image resize, shims for older IE’s and what not.
Then should we rewrite the upload method of the module? We don’t want to go down that rabbit hole. We should avoid modifying vendor libraries as much as possible. 

### Decorate It
Then our best option is to decorate the service, by adding/replacing methods during our module’s config phase.

Here is a simple upload example provided by the creator of ng-file-upload on its [GitHub page](https://github.com/danialfarid/ng-file-upload){:target="_blank"}.  
  
<br/>
**View**
{% highlight html %}

<body ng-app="fileUpload" ng-controller="MyCtrl">
  <form name="myForm">
 
    <fieldset>
 
      <legend>Upload on form submit</legend>
 
      Username:
      <input type="text" name="userName" ng-model="username" size="31" required>

      <i ng-show="myForm.userName.$error.required">*required</i>

      Photo:
      <input type="file" ngf-select ng-model="picFile" name="file"    
             accept="image/*" ngf-max-size="2MB" required>
 
      <i ng-show="myForm.file.$error.required">*required</i><br>
 
      <i ng-show="myForm.file.$error.maxSize">File too large 
          {picFile.size / 1000000|number:1}MB: max 2M</i>
 
      <img ng-show="myForm.file.$valid" ngf-thumbnail="picFile" class="thumb"> <button ng-click="picFile = null" ng-show="picFile">Remove</button>

 
      <button ng-disabled="!myForm.$valid" 
              ng-click="uploadPic(picFile)">Submit</button>
 
      <span class="progress" ng-show="picFile.progress >= 0">
        <div style="width:{{picFile.progress}}%" 
            ng-bind="picFile.progress + '%'"></div>
      </span>
 
      <span ng-show="picFile.result">Upload Successful</span>
      <span class="err" ng-show="errorMsg">{{errorMsg}}</span>
 
    </fieldset>

  </form>
</body>
{% endhighlight %}

<br/>
**Controller**
{% highlight js %}
//inject angular file upload directives and services.
var app = angular.module('fileUpload', ['ngFileUpload']);
 
app.controller('MyCtrl', ['$scope', 'Upload', '$timeout', function ($scope, Upload, $timeout) {
 
    $scope.uploadPic = function(file) {
    file.upload = Upload.upload({
      url: 'https://angular-file-upload-cors-srv.appspot.com/upload',
      data: {file: file, username: $scope.username},
    });
 
    file.upload.then(function (response) {
      $timeout(function () {
        file.result = response.data;
      });
    }, function (response) {
        $scope.errorMsg = response.status + ': ' + response.data;
    }), function (evt) {
      // Math.min is to fix IE which reports 200% sometimes
      file.progress = Math.min(100, parseInt(100.0 * evt.loaded / evt.total));
    });
    }
}]);
{% endhighlight %}

<br/>
### Replacing the upload method without touching vendors code
For the sake of the example, let’s say we’ve used his example as is. We also used the drag and drop, image resize and shim functionality.
But now we need to replace only the upload method without touching any code of the library. 
<br/>

#### Step 1 – Replacing the upload method of the service in our module’s config phase
{% highlight js %}
app.config(function($provide) {
    $provide.decorator('Upload', function($delegate, $q) {
 
        $delegate.upload = function(url, data) {
 
            var deferred = $q.defer();
 
            console.log('We have replaced the original upload method');
 
            deferred.resolve();
 
            // no error handling for the sake of simplicity
 
            return deferred.promise;
        }
 
        return $delegate;
    });
});
{% endhighlight %}

<br/>
#### Step 2 – Replace the .upload usage in our controller to play nicely with our new upload method
{% highlight js %}
file.upload = Upload.upload({
  url: 'https://angular-file-upload-cors-srv.appspot.com/upload',
  data: {file: file, username: $scope.username},
}).then(function(result) {
    $scope.picFile.result = true;
});
{% endhighlight %}

As you can see, in the config phase of our module, we used $provide to decorate our Upload service.
We have used $delegate (which is the original service instance) and replaced its upload method to a method that suits our needs.
Then we adjusted our controller to play nicely with our new .upload method, and that’s it.

In some cases, when we can make the replaced method to return the same data to success/errors, we don’t even have to touch our controllers.
So from a possibly costly change in the application, we can get away with decorating one single method.
