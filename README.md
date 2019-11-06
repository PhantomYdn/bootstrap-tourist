# Bootstrap Tourist [![NPM Version](http://img.shields.io/npm/v/bootstrap-tourist.svg?style=flat)](https://www.npmjs.org/) [![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org)

Quick and easy way to build your product tours with Bootstrap Popovers for Bootstrap 3 and 4.

## About Bootstrap Tourist
Bootstrap Tourist (called "Tourist" from here on) is a built upon [Bootstrap Tour](https://github.com/sorich87/bootstrap-tour), a plugin to create product tours.

Bootstrap Tour was written in coffeescript, and had a number of open feature and bug fix requests in the github repo. Bootstrap Tourist is an in-progress effort to move Bootstrap Tour to native ES6, fix some issues and add some requested features. You can read more about why Bootstrap Tourist exists, and why it's not a github fork anymore, [here](https://github.com/sorich87/bootstrap-tour/issues/713)

Tourist works with Bootstrap 3.4 and 4.3 (specify "framework" option), however the "standalone" non-Bootstrap version is not available

**NOTE**: A minified version is not provided because the entire purpose of this repo is to enable fixes, features and native port to ES6. If you are uncomfortable with this, please either use the original Bootstrap Tour, or minify the files yourself!

## Minimum Bootstrap / jQuery requirements
Please use at a minimum Bootstrap 3.4.x or 4.3.x, and jQuery 3.3.1, as there are some bugs in earlier versions that could cause problems with Tourist. Some earlier versions may work, but are not officially supported.

## Quick Start

### Fresh Start
If you are new to Tourist, and don't have Bootstrap Tour working, it's as simple as doing the following:
1. First include `bootstrap-tourist.js` and `bootstrap-tourist.css` on your page, after jQuery and Bootstrap, something like:
    ```html
    <link href="bootstrap.min.css" rel="stylesheet">
    <link href="bootstrap-tourist.css" rel="stylesheet">

    ...

    <script src="jquery.min.js"></script>
    <script src="bootstrap.min.js"></script>
    <script src="bootstrap-tourist.js"></script>
    ```
1. Next, initialize your tour with some steps, and then start it:
    ```js
    var tour = new Tour({
        framework: "bootstrap3",   // or "bootstrap4" depending on your version of bootstrap
        steps: [
            {
                element: "#my-element",
                title: "Title of my step",
                content: "Content of my step"
            },
            {
                element: "#my-other-element",
                title: "Title of my step",
                content: "Content of my step"
            }
        ]
    });

    // Start the tour
    // **IMPORTANT!** - don't need to call `tour.init()`
    tour.start();
    ```

### Replacing Boostrap Tour
If you already have a working tour using Bootstrap Tour, and you want to move to Tourist:
1. Replace the Bootstrap Tour JS and CSS with the Tourist ones:
    ```html
    <link href="bootstrap-tourist.css" rel="stylesheet">

    ...

    <script src="bootstrap-tourist.js"></script>
    ```
1. If you are using Bootstrap 4, add the `framework: "bootstrap4"` option to your initialization code:
    ```js
    var tour = new Tour({
        debug: true,               // you may wish to turn on debug for the first run through
        framework: "bootstrap4",   // set Tourist to use BS4 compatibility
        steps: [...steps go here...],
    });
    ```
1. Remove the call to `tour.init()` - this is no longer required
1. Make sure there's a call to `tour.start()` to start the tour, and optionally add a call to `tour.restart()` to force restart the tour

## Documentation
Tourist now has further documentation included in the repo under the `/docs/` folder. Take a look!
**IMPORTANT!** - Steps linked to elements ALWAYS stay stuck to their element, even if the user scrolls the element & tour popover off the screen.
**NOTE** - For [BootstrapDialog plugin](https://nakupanda.github.io/bootstrap3-dialog/), since it creates random UUIDs for the dialog DOM ID. You need to fix the ID to something you know. Do the following:
    ```js
    // Setup BoostrapDialog
    var boostrapDialog = new BootstrapDialog.confirm({
        ...options
    });

    // BootstrapDialog gives a random GUID ID for dialog. Give it a proper one
    var $modal = boostrapDialog.getModal();
    $modal.attr('id', 'myModal');

    boostrapDialog.setId('myModal');

    // Now use `myModal` element in a tour step
    var tour = new Tour({
        steps: [
            {
                element: '#myModal'
            }
        ]
    });

    tour.start();
    ```

### Global

#### `framework`
Indicates the framework that's being used, which determines the default `template` to use, as well as changes which methods are used to manage the Tour popovers to be either BS3 or BS4 compatible (DEFAULT `bootstrap3`)
```js
var tour = new Tour({
    framework: 'bootstrap3' // or "bootstrap4" depending on your version of bootstrap
});
```
**IMPORTANT!** - Only `bootstrap3` and `bootstrap4` are valid options by default
**NOTE** - To add additional custom framework templates, search the code for `PLACEHOLDER: TEMPLATES LOCATION`. There is an array that contains the templates, simply edit or add as required.

#### `template`
The popover template to use when rendering the tour on the page (DEFAULT `null`)
```js
var tour = new Tour({
    template: '<div class="popover" role="tooltip">....blah....</div>'
});
```
**NOTE** - If not specified, or when set to `null`, Tourist uses the `framework`'s default popover template

#### `onPreviouslyEnded()`
Callback for if `tour.start()` is called on a tour that has previously ended
```js
var tour = new Tour({
    onPreviouslyEnded: function (tour) {
        console.log('Looks like this tour has already ended');
    }
});

tour.start();
```

#### `sanitizeWhitelist`
BS 3.4.1 added a sanitizer to popovers and tooltips - this breaking change strips non-whitelisted DOM elements from popover content, title etc. To prevent future similar re-occurrences, and also allow the manipulation of the sanitizer "allowed list". To understand the purpose and operation of these features, review the [Bootstrap sanitizer](https://getbootstrap.com/docs/3.4/javascript/#js-sanitizer)
An object with the same structure as the default Bootstrap whitelist, that will be merged it.
```js
var tour = new Tour({
    sanitizeWhitelist:  {
        button: ['data-someplugin1', 'data-somethingelse'], // allows <button data-someplugin1="abc", data-somethingelse="xyz">
        select: []  // allows <select>
    }
});
```

#### `sanitizeFunction()`
A function that will be used to sanitize Tour content.
```js
var tour = new Tour({
    sanitizeFunction: function(stepContent) {
        // Bypass Bootstrap sanitizer using custom function to clean the tour step content.
        // stepContent will contain the content of the step, i.e.: tourSteps[n].content. You must
        // clean this content to prevent XSS and other vulnerabilities. Use your own code or a lib like DOMPurify
        return DOMPurify.sanitize(stepContent);
    }
});
```
**IMPORTANT!** - Specifying a function for this option will cause `sanitizeWhitelist` to be ignored. However, specifying anything other than a function will cause `sanitizeWhitelist` to be used.
**NOTE** - If you have complete control over the tour content (ie. no risk of XSS or similar attacks), you can use `sanitizeFunction` to bypass all sanitization and use your step content exactly as is by simply returning the content:
    ```js
    var tour = new Tour({
        sanitizeFunction: function (stepContent) {
            // POTENTIAL SECURITY RISK
            // bypass Bootstrap sanitizer, perform no sanitization, tour step content will be exactly as templated in tourSteps.
            return stepContent;
        }
    });
    ```

#### `localization`

##### `buttonTexts`
Change the text displayed for the buttons used in the tour step popovers
```js
var tour = new Tour({
    localization: {
        buttonTexts: {
            prevButton: 'Back',
            nextButton: 'Next',
            pauseButton: 'Wait',
            resumeButton: 'Continue',
            endTourButton: 'Ok, enough'
        }
    }
});
```
**IMPORTANT!** - This option only applies to the default templates

### Per-step

#### `element`
The element that is being focused per step, can either be a string selector, jQuery element, or a function that returns a DOM element
```js
var tour = new Tour({
    step: [
        {
            // Use a function that returns a jQuery element
            element: function () {
                return $(document).find('.something');
            },
            title: 'Function',
            content: 'Element found by function'
        },
        {
            element: '#selector',   // Use a string selector
            title: 'Selector',
            content: 'Element found with selector string'
        },
        {
            element: $('#element'), // Use a jQuery object/element
            title: 'jQuery Element',
            content: 'Element found by jQuery'
        }
    ]
});
```

#### `reflexOnly`
Hide the _Next_ button in the tour until the user clicks the element (DEFAULT `false`)
```js
var tour = new Tour({
    steps: [
        {
            element: '#myButton',
            reflex: true,
            reflexOnly: true,
            title: 'Click it',
            content: "Click to continue, or you're stuck"
        }
    ]
})
```

#### `preventInteraction`
Prevent interaction with a step's element (DEFAULT `false`)
```js
var tour = new Tour({
    steps: [
        {
            element: '#btnMCHammer',
            title: 'Hammer Time',
            content: "You can't touch this",
            preventInteraction: true
        }
    ]
});
```

#### `delayOnElement`
Wait for an element to appear before continuing tour (DEFAULT `null`)

##### `delayElement`
The element to wait for, could be the step's element (`element`), string selector, jQuery element, or a function that returns a DOM element
 ```js
var tour = new Tour({
    steps: [
        {
            element: '#btnPrettyTransition',
            title: 'Ages',
            content: 'This button takes ages to appear',
            delayOnElement: {
                delayElement: 'element' // use string 'element' to wait for this step's element (ie. '#btnPrettyTransition')
            }
        },
        {
            element: '#btnAnnotherPrettyTransition',
            title: 'Function',
            content: "This button takes ages to appear, we're waiting for an element returned by function",
            delayOnElement: {
                // Use a function that returns a DOM element
                delayElement: function () {
                    return $('#btnAnnotherPrettyTransition');
                }
            }
        },
        {
            element: '#inputUnrelated',
            title: 'Waiting',
            content: 'This input is nice, but you only see this step when the other div appears',
            delayOnElement: {
                delayElement: '#divStuff' // Use a string selector
            }
        },
        {
            element: '#btnElement',
            title: 'Function',
            content: "This button doesn't work nice unless #element appears",
            delayOnElement: {
                delayElement: $('#element'), // Use a jQuery object/element
            }
        }
    ]
});
```

##### `maxDelay`
The maximum amount of time, in milliseconds to wait for the element to appear (DEFAULT `2000`, ie. 2 seconds)
```js
var tour = new Tour({
    steps: [
        {
            element: '#btnDontForgetThis',
            title: "Cool",
            content: "Remember the 'onElementUnavailable' option!",
            delayOnElement: {
                delayElement: $('#element'), // Use a jQuery object/element
                maxDelay: 5000  // Wait a maximum of 5 seconds
            },
            // This will be called if $('#element') is not visible after 5 seconds
            onElementUnavailable: function (tour, stepNumber) {
                console.log('Well that went badly!');
            }
        }
    ]
});
```

### Either global or per-step

#### `onNext()` & `onPrevious()`
Control flow options, which are available per step or globally
```js
var tour = new Tour({
    steps: [
        {
            element: '#inputBanana',
            title: 'Bananas!',
            content: "Bananas are yellow, except when they're not",
            onNext: function (tour) {
                if ($('#inputBanana').val() !== 'banana') {
                    $('#inputBanana').css('background-color', 'red');

                    // Returning false will prevent Tour from automatically moving to the next/previous step.
                    return false;
                }
            }
        }
    ],
    onPrevious: function (tour) {
        if (true) {
            // Tour flow methods (`goTo()`, etc) now also work correctly in `onNext()`/`onPrevious()``
            tour.goTo(3);   // Force the tour to jump to step 3

            // **IMPORTANT!** - Prevent default move to next step
            return false;
        }
    }
})
```

#### `onElementUnavailable`
When a step's element does not exist (DEFAULT `null`)
```js
var tour = new Tour({
    steps: [
        {
            element: '#btnThis',
            title: 'Some button',
            content: 'Some content'
        },
        {
            element: '#btnThat',
            title: 'Another button',
            content: 'More content',
            // Override the default handler for this step only
            onElementUnavailable: function (tour, stepNumber) {
                alert('The tour broke on #btnThat step');

                tour.goTo(1);

                return false;
            }
        }
    ],
    // "Unavailable Element" handler for all tour steps
    onElementUnavailable: function (tour, stepNumber) {
        alert('The default error handler: tour element is done broke on step number ' + stepNumber);
    }
});
```
**IMPORTANT!** - orphan steps are stuck to the center of the screen

#### `showIfUnintendedOrphan`
To show a tour step as an orphan if its element doesn't exist, overriding `onElementUnavailable` (DEFAULT `false`)
```js
var tour = new Tour({
    steps: [
        {
            element: '#btnSomething',
            title: 'Always',
            content: "This tour step will always appear, either against element '#btnSomething' if it exists, or as an orphan if it doesn't"
        },
        {
            element: '#btnSomethingElse',
            title: 'Always after a delay',
            content: "This tour step will always appear. If element '#btnSomethingElse' doesn't exist, delayOnElement will wait until it exists. If delayOnElement times out, step will show as an orphan",
            showIfUnintendedOrphan: true,
            delayOnElement: {
                delayElement: 'element' // Use string 'element' to wait for this step's element, ie. '#btnSomethingElse'
            }
        },
        {
            element: '#btnDoesntExist',
            title: 'Always',
            content: 'This tour step will always appear',
            showIfUnintendedOrphan: true,
            onElementUnavailable: function () {
                console.log('This will never get called as `showIfUnintendedOrphan` will show step as an orphan');
            }
        },
        {
            element: '#btnDoesntExistEither',
            title: 'Never',
            content: "You'll never see this",
            showIfUnintendedOrphan: false
        }
    ],
    showIfUnintendedOrphan: true
});
```
**IMPORTANT!** - `delayOnElement` takes priority over this option, so the delay will timeout before the step will be shown as an orphan.

#### `onModalHidden`
When a tour element is a, or within a modal, the element may disappear if the user dismisses the dialog (DEFAULT `null`)
```js
var tour = new Tour({
    steps: [
        {
            onModalHidden: function (tour, stepNumber) {
                // Jump to step 3
                return 3;
            }
        }
    ],
    onModalHidden: function (tour, stepNumber) {
        if (true) {
            // Remain on the current step
            return false;
        }

        // Continue to next step
        // Not returning anything will do the same
        return null;
    }
});
```
**IMPORTANT!** - Only works when step is not orphaned
**NOTE** - Different return values will cause different actions:
    - `int` step number to immediately move to that step
    - `false` to stay on the current step
    - `null` or nothing at all to move to the next step

#### `showProgressBar` & `showProgressText`
Show the progress bar & progress text (DEFAULT `true`)
```js
var tour = new Tour({
    steps: [
        {
            element: '#inputBanana',
            title: 'Bananas!',
            content: "Bananas are yellow, except when they're not",
        },
        {
            element: '#inputOranges',
            title: 'Oranges!',
            content: 'Oranges are not bananas',
            showProgressBar: false,     // Do not show the progress bar on this step only
            showProgressText: false,    // Do not show the progress text on this step only
        }
    ],
    showProgressBar: true,      // default show progress bar
    showProgressText: true,     // default show progress text
});
```

#### `getProgressBarHTML()` & `getProgressTextHTML()`
Customize the progress bar & progress text (DEFAULT `null`)
```js
const tour = new Tour({
    steps: [
        {
            element: '#inputBanana',
            title: 'Bananas!',
            content: "Bananas are yellow, except when they're not",
        },
        {
            element: '#inputOranges',
            title: 'Oranges!',
            content: 'Oranges are not bananas',
            // Override the default handler for this step only
            getProgressBarHTML: function (percent) {
                return "<div>You're " + percent + " of the way through!</div>";
            }
        }
    ],
    showProgressBar: true,
    showProgressText: true,
    // "Progress Bar" handler for all tour steps
    getProgressBarHTML: function (percent) {
        // Return valid HTML to add a custom progress bar
        return '<div class="progress"><div class="progress-bar progress-bar-striped" role="progressbar" style="width: ' + percent + '%;"></div></div>';
    },
    // "Progress Text" handler for all tour steps
    getProgressTextHTML: function (stepNumber, percent, stepCount) {
        return 'Slide ' + stepNumber + '/' + stepCount;
    }
});
```

#### `backdropOptions`
Overlay divs and customizable transitions between tour steps

##### `highlightColor`
The _HEX_ color code for the highlight. (DEFAULT `#FFF`)

##### `highlightOpacity`
The _Alpha_ value of the div used to highlight the step element (DEFAULT `0.9`)

##### `animation`
The animations that happen between different states, for all of the following sub-options, either a function, or a CSS class string can be specified.
**NOTE** - When providing a CSS class, this class will be applied to the backdrop/highlight element at the specified time and removed once the transition is complete.
    For example, assume you create a class in your CSS that animates an effect as follows:
        ```css
        .my-custom-animation {
            -webkit-transition: all .5s ease-out;
            -moz-transition: all .5s ease-out;
            -ms-transition: all .5s ease-out;
            -o-transition: all .5s ease-out;
            transition: all .5s ease-out;
        }
        ```
    You can then use this effect every time the background overlay div is shown by specifying it as follows:
        ```js
        var tour = new Tour({
            backdropOptions: {
                animation: {
                    backdropShow: "my-custom-animation"
                }
            }
        });
        ```
    Now, when moving between a step without a backdrop to one that has a backdrop, your class will be used to implement the transition, in the following manner:
        - The class will be added before the backdrop is shown and removed when the transition is complete.
    In other words, specifying a CSS class for `backdropShow` is functionally equivalent to the following code executed when the tour moves between steps:
        ```js
        $(backdropOptions element).addClass("my-custom-animation");
        $(backdropOptions element).show(0, function() {
            $(this).removeClass("my-custom-animation");
        });
        ```
    **IMPORTANT!** - The CSS class is removed after the transition is complete, therefore only use this to apply CSS transitions - do not use it for "persistent" CSS changes
**NOTE** - When providing a function, it must position and show the highlight/backdrop element, and must have the following signature `function (domElement, step)`.
    The `domElement` parameter provides a jQuery object for the element you must manipulate.
        - For example, if you have specified a function for `backdropOptions.animation.highlightShow`, then domElement will be the highlighting div. You must then correctly position and show this div over the step element.
    The `step` parameter provides an object with information about the step. It has the following props, most of which are taken from your tour step and re-provided to you for ease:
        ```js
        step = {
            element,                // the actual step element, even if the tour uses a function for this step.element option
            container,              // the container option (string) as specified in the step or globally, to help you decide how to set up the transition
            backdrop,               // as per step option (bool)
            preventInteraction,     // as per step option (bool)
            isOrphan,               // whether is step is actually an orphan, because the element was wrong and showIfUnintendedOrphan == true or for some other reason (bool)
            orphan,                 // as per step option (bool)
            showIfUnintendedOrphan, // as per step option (bool)
            duration,               // as per step option (bool)
            delay,                  // as per step option (bool)
            fnPositionHighlight     // a helper function to position the highlight element (SEE BELOW FOR MORE INFO)
        };
        ```
        - The step parameter's `fnPositionHighlight` option is provided to make it easy for you to automatically position the highlight div on top of the tour step element.
            - The function simply performs the following:
                ```js
                function fnPositionHighlight() {
                    $(DOMID_HIGHLIGHT).width(_stepElement.outerWidth())
                        .height(_stepElement.outerHeight())
                        .offset(_stepElement.offset());
                }
                ```
            - This allows you to do the following in your customized transition code:
                ```js
                function (domElement, step) {
                    // do whatever setup for your custom transition
                    step.fnPositionHighlight();
                }
                ```

    **IMPORTANT!** - The provided function is 100% responsible for both positioning AND showing the highlight/backdrop element

###### `backdropShow`
Animation for when a previously hidden backdrop is shown (DEFAULT shown below)
```js
var tour = new Tour({
    backdropOptions: {
        animation: {
            backdropShow: function (domElement, step) {
                domElement.fadeIn();
            }
        }
    }
});
```

###### `backdropHide`
Animation for when a previously visible backdrop is hidden (DEFAULT shown below)
```js
var tour = new Tour({
    backdropOptions: {
        animation: {
            backdropHide: function (domElement, step) {
                domElement.fadeOut('slow');
            }
        }
    }
});
```

###### `highlightShow`
Animation for when step N did not have an element, and step N+1 does have an element (DEFAULT shown below)
```js
var tour = new Tour({
    backdropOptions: {
        animation: {
            highlightShow: function (domElement, step) {
                // Calling step.fnPositionHighlight() is the same as:
                // domElement.width($(step.element).outerWidth()).height($(step.element).outerHeight()).offset($(step.element).offset());
                step.fnPositionHighlight();

                domElement.fadeIn();
            }
        }
    }
});
```

###### `highlightTransition`
Animation for when step N has an element, and step N+1 does not have an element (DEFAULT shown below)
```js
var tour = new Tour({
    backdropOptions: {
        animation: {
            highlightTransition: 'tour-highlight-animation'
        }
    }
});
```

###### `highlightHide`
Animation for when both step N and step N+1 have an element, and the highlight is visibly moved from one to the other (DEFAULT shown below)
```js
var tour = new Tour({
    backdropOptions: {
        animation: {
            highlightHide: function (domElement, step) {
                domElement.fadeOut('slow');
            }
        }
    }
});
```

### All options
With defaults

```js
var tour = new Tour({
    name: 'tour',
    steps: [],
    container: 'body',
    autoscroll: true,
    keyboard: true,
    storage: window.localStorage || false,
    debug: false,
    backdrop: false,
    backdropContainer: 'body',
    backdropOptions: {
        highlightOpacity: 0.9,
        highlightColor: '#FFF',
        animation: {
            // can be string of css class or function signature: function(domElement, step) {}
            backdropShow: function (domElement, step) {
                domElement.fadeIn();
            },
            backdropHide: function (domElement, step) {
                domElement.fadeOut('slow')
            },
            highlightShow: function (domElement, step) {
                // calling step.fnPositionHighlight() is the same as:
                // domElement.width($(step.element).outerWidth()).height($(step.element).outerHeight()).offset($(step.element).offset());
                step.fnPositionHighlight();
                domElement.fadeIn();
            },
            highlightTransition: 'tour-highlight-animation',
            highlightHide: function (domElement, step) {
                domElement.fadeOut('slow')
            }
        },
    },
    redirect: true,
    orphan: false,
    showIfUnintendedOrphan: false,
    duration: false,
    delay: false,
    basePath: '',
    template: null,
    localization: {
        buttonTexts: {
            prevButton: 'Prev',
            nextButton: 'Next',
            pauseButton: 'Pause',
            resumeButton: 'Resume',
            endTourButton: 'End Tour'
        }
    },
    framework: 'bootstrap3',
    sanitizeWhitelist: [],
    sanitizeFunction: null,                 // function(content) {}
    showProgressBar: true,
    showProgressText: true,
    getProgressBarHTML: null,               // function(percent) {},
    getProgressTextHTML: null,              // function(stepNumber, percent, stepCount) {},
    afterSetState: function (key, value) {},
    afterGetState: function (key, value) {},
    afterRemoveState: function (key) {},
    onStart: function (tour) {},
    onEnd: function (tour) {},
    onShow: function (tour) {},
    onShown: function (tour) {},
    onHide: function (tour) {},
    onHidden: function (tour) {},
    onNext: function (tour) {},
    onPrev: function (tour) {},
    onPause: function (tour, duration) {},
    onResume: function (tour, duration) {},
    onRedirectError: function (tour) {},
    onElementUnavailable: null,             // function (tour, stepNumber) {}
    onPreviouslyEnded: null,                // function (tour) {}
    onModalHidden: null                     // function(tour, stepNumber) {}
});
```

## Policy on NPM, semantic versioning, releases, code structure etc
I'm a self-taught C++/x86 asm coder from the 80's and 90's. I'm not a web developer, I only have basic html, js, jquery knowledge that I pretty much taught myself over a couple of weeks or so. This isn't my full time job, or even my part time job. I inherited the need to fix some stuff in Bootstrap Tour, and I simply published my fixes on github. I never intended to take on maintenance of a product, or become responsible for it in any way. I even said this in the Tour repo when I published my first fixes.

All of that info is to set your expectations. Tourist works, and it's been thoroughly tested, but I keep getting tripped up by the niceties of modern coding. Tourist is offered as a non-minified, simple download-and-drop-in tool for you to use if you want to. I will do my best to follow coding standards, publish to npm when I remember, keep to a coding style, follow semantic versioning and all that stuff that makes it easy for you to use this plugin. However please fully expect that:
1. I will probably fail, so you'll need to tell me what I've done wrong and most importantly how to fix it
1. The github repo will always have the latest in-progress version
1. The github release will always be the latest stable version
1. Anything else is a bonus.

As a side note, Bootstrap Tour was made in coffeescript (which I'd never heard of). So when I started working on Tourist, it was using a codebase without comments, odd transpilation structures and approaches, and much more. So if you're looking at the source and scratching your head as to why something is done in a certain way - yes, welcome to my world :-)

## Contributing
Feel free to contribute with pull requests, bug reports or enhancement suggestions.

## License
Code licensed under the [MIT license](https://opensource.org/licenses/MIT). Documentation licensed under [CC BY 3.0](http://creativecommons.org/licenses/by/3.0/).
