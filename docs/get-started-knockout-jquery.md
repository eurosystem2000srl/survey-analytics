# Add Survey Analytics to a Knockout or jQuery Application

This step-by-step tutorial will help you get started with Survey Analytics in a Knockout or jQuery application. To add Survey Analytics to your application, follow the steps below:

- [Link Resources](#link-resources)
- [Load Survey Results](#load-survey-results)
- [Configure the Visualization Panel](#configure-the-visualization-panel)
- [Render the Visualization Panel](#render-the-visualization-panel)

As a result, you will create the following dashboard:

<p class="codepen" data-height="532.7999572753906" data-default-tab="js,result" data-slug-hash="vYdmvgz" data-user="romantsukanov" style="height: 532.7999572753906px; box-sizing: border-box; display: flex; align-items: center; justify-content: center; border: 2px solid; margin: 1em 0; padding: 1em;">
  <span>See the Pen <a href="https://codepen.io/romantsukanov/pen/vYdmvgz">
  SurveyJS - Add Survey Analytics to a Knockout or jQuery Application</a> by RomanTsukanov (<a href="https://codepen.io/romantsukanov">@romantsukanov</a>)
  on <a href="https://codepen.io">CodePen</a>.</span>
</p>
<script async src="https://cpwebassets.codepen.io/assets/embed/ei.js"></script>

Survey Analytics is powered by Knockout and does not have an individual implementation for jQuery. However, you can integrate the version for Knockout into your jQuery application by following the same instructions.

You can find the full code for this tutorial in the following GitHub repository: <a href="https://github.com/surveyjs/code-examples/tree/main/get-started-analytics/knockout" target="_blank">Get Started with SurveyJS Analytics - Knockout</a>.

## Link Resources

Survey Analytics depends on other JavaScript libraries. Reference them on your page in the following order:

- **Knockout**

- **Survey Core**       
A platform-independent part of the SurveyJS Library that works with the survey model. Survey Analytics requires only this part, but if you also display the survey on the page, reference [the rest of the SurveyJS Library resources](/Documentation/Library?id=get-started-knockout#link-surveyjs-resources) as well.

- **<a href="https://github.com/plotly/plotly.js#readme" target="_blank">Plotly.js</a> and <a href="https://github.com/timdream/wordcloud2.js#readme" target="_blank">Wordcloud</a>**       
Wordcloud (optional) is used to visualize the Text, Multiple Text, and Comment question types. Plotly.js (required) is used to visualize the rest of the question types.

- **Survey Analytics**      
A library that integrates Survey Core with Plotly.js and Wordcloud.

The following code shows how to reference these libraries:

```html
<head>
    <!-- ... -->
    <script type="text/javascript" src="https://unpkg.com/knockout/build/output/knockout-latest.js"></script>

    <!-- SurveyJS Library resources -->
    <script type="text/javascript" src="https://unpkg.com/survey-core/survey.core.min.js"></script>
    <!-- Uncomment the following lines if you also display the survey on the page -->
    <!-- <link href="https://unpkg.com/survey-core/modern.min.css" type="text/css" rel="stylesheet"> -->
    <!-- <script type="text/javascript" src="https://unpkg.com/survey-knockout-ui/survey-knockout-ui.min.js"></script> -->

    <!-- Third-party visualization libraries -->
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    <!-- Uncomment the following line if you use the Text, Multiple Text, or Comment question types in your surveys -->
    <!-- <script src="https://unpkg.com/wordcloud/src/wordcloud2.js"></script> -->

    <!-- Survey Analytics resources-->
    <link href="https://unpkg.com/survey-analytics/survey.analytics.min.css" rel="stylesheet">
    <script src="https://unpkg.com/survey-analytics/survey.analytics.min.js"></script>
</head>
```

## Load Survey Results

You can access survey results as a JSON object within the `SurveyModel`'s `onComplete` event handler. Send the results to your server and store them with a specific survey ID. Refer to the [Handle Survey Completion](/Documentation/Library?id=get-started-knockout#handle-survey-completion) help topic for more information.  

To load the survey results, send the survey ID to your server and return an array of JSON objects:

```js
const SURVEY_ID = 1;

loadSurveyResults("https://your-web-service.com/" + SURVEY_ID)
    .then((surveyResults) => {
        // ...
        // Configure and render the Visualization Panel here
        // Refer to the help topics below
        // ...
    });

function loadSurveyResults (url) {
    return new Promise((resolve, reject) => {
        const request = new XMLHttpRequest();
        request.open('GET', url);
        request.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
        request.onload = () => {
            const response = request.response ? JSON.parse(request.response) : [];
            resolve(response);
        }
        request.onerror = () => {
            reject(request.statusText);
        }
        request.send();
    });
}
```

For demonstration purposes, this tutorial uses predefined survey results. The following code shows a survey model and the structure of the survey results array:

```js
const surveyJson = {
    elements: [{
        name: "satisfaction-score",
        title: "How would you describe your experience with our product?",
        type: "radiogroup",
        choices: [
            { value: 5, text: "Fully satisfying" },
            { value: 4, text: "Generally satisfying" },
            { value: 3, text: "Neutral" },
            { value: 2, text: "Rather unsatisfying" },
            { value: 1, text: "Not satisfying at all" }
        ],
        isRequired: true
    }, {
        name: "nps-score",
        title: "On a scale of zero to ten, how likely are you to recommend our product to a friend or colleague?",
        type: "rating",
        rateMin: 0,
        rateMax: 10,
    }],
    showQuestionNumbers: "off",
    completedHtml: "Thank you for your feedback!",
};

const surveyResults = [{
    "satisfaction-score": 5,
    "nps-score": 10
}, {
    "satisfaction-score": 5,
    "nps-score": 9
}, {
    "satisfaction-score": 3,
    "nps-score": 6
}, {
    "satisfaction-score": 3,
    "nps-score": 6
}, {
    "satisfaction-score": 2,
    "nps-score": 3
}];
```

## Configure the Visualization Panel

Analytics charts are displayed in a Visualization Panel. Specify [its properties](/Documentation/Analytics?id=ivisualizationpaneloptions) in a configuration object. In this tutorial, the object enables the [`allowHideQuestions`](/Documentation/Analytics?id=ivisualizationpaneloptions#allowHideQuestions) property:

```js
const vizPanelOptions = {
    allowHideQuestions: false
}
```

Pass the configuration object, survey questions, and results to the `VisualizationPanel` constructor as shown in the code below to instantiate the Visualization Panel. Assign the produced instance to a constant that will be used later to render the component:

```js
const surveyJson = { /* ... */ };
const surveyResults = [ /* ... */ ];
const vizPanelOptions = { /* ... */ };

const survey = new Survey.Model(surveyJson);

const vizPanel = new SurveyAnalytics.VisualizationPanel(
    survey.getAllQuestions(),
    surveyResults,
    vizPanelOptions
);
```

<details>
    <summary>View full code</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>SurveyJS Analytics for Knockout</title>
    <meta charset="utf-8">
    <script type="text/javascript" src="https://unpkg.com/knockout/build/output/knockout-latest.js"></script>

    <script type="text/javascript" src="https://unpkg.com/survey-core/survey.core.min.js"></script>
    <!-- Uncomment the following lines if you also display the survey on the page -->
    <!-- <link href="https://unpkg.com/survey-core/modern.min.css" type="text/css" rel="stylesheet"> -->
    <!-- <script type="text/javascript" src="https://unpkg.com/survey-knockout-ui/survey-knockout-ui.min.js"></script> -->

    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    <!-- Uncomment the following line if you use the Text, Multiple Text, or Comment question types in your surveys -->
    <!-- <script src="https://unpkg.com/wordcloud/src/wordcloud2.js"></script> -->

    <link href="https://unpkg.com/survey-analytics/survey.analytics.min.css" rel="stylesheet">
    <script src="https://unpkg.com/survey-analytics/survey.analytics.min.js"></script>

    <script type="text/javascript" src="index.js"></script>
</head>
<body>

</body>
</html>
```

```js
// Uncomment the following line if you also display the survey on the page
// Survey.StylesManager.applyTheme("modern");

const surveyJson = {
    elements: [{
        name: "satisfaction-score",
        title: "How would you describe your experience with our product?",
        type: "radiogroup",
        choices: [
            { value: 5, text: "Fully satisfying" },
            { value: 4, text: "Generally satisfying" },
            { value: 3, text: "Neutral" },
            { value: 2, text: "Rather unsatisfying" },
            { value: 1, text: "Not satisfying at all" }
        ],
        isRequired: true
    }, {
        name: "nps-score",
        title: "On a scale of zero to ten, how likely are you to recommend our product to a friend or colleague?",
        type: "rating",
        rateMin: 0,
        rateMax: 10,
    }],
    showQuestionNumbers: "off",
    completedHtml: "Thank you for your feedback!",
};

const survey = new Survey.Model(surveyJson);

const surveyResults = [{
    "satisfaction-score": 5,
    "nps-score": 10
}, {
    "satisfaction-score": 5,
    "nps-score": 9
}, {
    "satisfaction-score": 3,
    "nps-score": 6
}, {
    "satisfaction-score": 3,
    "nps-score": 6
}, {
    "satisfaction-score": 2,
    "nps-score": 3
}];

const vizPanelOptions = {
    allowHideQuestions: false
}

const vizPanel = new SurveyAnalytics.VisualizationPanel(
    survey.getAllQuestions(),
    surveyResults,
    vizPanelOptions
);
```
</details>

## Render the Visualization Panel

Switch to the component template. Add a page element that will serve as the Visualization Panel container:

```html
<body>
    <div id="surveyVizPanel"></div>
</body>
```

To render the Visualization Panel in the page element, call the `render(containerId)` method on the Visualization Panel instance you created in the previous step:

```js
document.addEventListener("DOMContentLoaded", function() {
    vizPanel.render("surveyVizPanel");
});
```

<details>
    <summary>View full code</summary>

```html
<!DOCTYPE html>
<html>
<head>
    <title>SurveyJS Analytics for Knockout</title>
    <meta charset="utf-8">
    <script type="text/javascript" src="https://unpkg.com/knockout/build/output/knockout-latest.js"></script>

    <script type="text/javascript" src="https://unpkg.com/survey-core/survey.core.min.js"></script>
    <!-- Uncomment the following lines if you also display the survey on the page -->
    <!-- <link href="https://unpkg.com/survey-core/modern.min.css" type="text/css" rel="stylesheet"> -->
    <!-- <script type="text/javascript" src="https://unpkg.com/survey-knockout-ui/survey-knockout-ui.min.js"></script> -->

    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
    <!-- Uncomment the following line if you use the Text, Multiple Text, or Comment question types in your surveys -->
    <!-- <script src="https://unpkg.com/wordcloud/src/wordcloud2.js"></script> -->

    <link href="https://unpkg.com/survey-analytics/survey.analytics.min.css" rel="stylesheet">
    <script src="https://unpkg.com/survey-analytics/survey.analytics.min.js"></script>

    <script type="text/javascript" src="index.js"></script>
</head>
<body>
    <div id="surveyVizPanel"></div>
</body>
</html>
```

```js
// Uncomment the following line if you also display the survey on the page
// Survey.StylesManager.applyTheme("modern");

const surveyJson = {
    elements: [{
        name: "satisfaction-score",
        title: "How would you describe your experience with our product?",
        type: "radiogroup",
        choices: [
            { value: 5, text: "Fully satisfying" },
            { value: 4, text: "Generally satisfying" },
            { value: 3, text: "Neutral" },
            { value: 2, text: "Rather unsatisfying" },
            { value: 1, text: "Not satisfying at all" }
        ],
        isRequired: true
    }, {
        name: "nps-score",
        title: "On a scale of zero to ten, how likely are you to recommend our product to a friend or colleague?",
        type: "rating",
        rateMin: 0,
        rateMax: 10,
    }],
    showQuestionNumbers: "off",
    completedHtml: "Thank you for your feedback!",
};

const survey = new Survey.Model(surveyJson);

const surveyResults = [{
    "satisfaction-score": 5,
    "nps-score": 10
}, {
    "satisfaction-score": 5,
    "nps-score": 9
}, {
    "satisfaction-score": 3,
    "nps-score": 6
}, {
    "satisfaction-score": 3,
    "nps-score": 6
}, {
    "satisfaction-score": 2,
    "nps-score": 3
}];

const vizPanelOptions = {
    allowHideQuestions: false
}

const vizPanel = new SurveyAnalytics.VisualizationPanel(
    survey.getAllQuestions(),
    surveyResults,
    vizPanelOptions
);

document.addEventListener("DOMContentLoaded", function() {
    vizPanel.render("surveyVizPanel");
});
```
</details>

<a href="https://github.com/surveyjs/code-examples/tree/main/get-started-analytics/knockout" target="_blank">View full code on GitHub</a>


## Further Reading

- [Analytics Demo Examples](/Examples/Analytics)