# Robobot API Developer Documentation
Read the documentation for the Robobot API carefully if you plan to implement conversational form models in your application.

## Table of Contents
1. [Introduction](#introduction)
2. [Basics](#basics)
    1. [HTTP Methods](#http-methods)
    2. [JSON](#json)
    3. [Authentication](#introduction)
3. [API Reference](#api-reference)
    1. [Access Code: Request](#access-code-request)
    2. [Form: Request](#form-request)
    3. [Next: Request](#next-request)
    4. [Next: Response Types](#response-types)
    5. [Report](#report)
4. [Local Cache Invalidation](#local-cache-invalidation)

## Introduction
Robobot aims to deliver a developer friendly approach for integrating conversational forms, created through the Robobot Dashboard, in existing software products.

## Basics

### HTTP Methods
Robobot API can be integrated easily with any existing HTTP client using a variety of programming languages and frameworks. The API currently only supports POST requests that need some headers for authentication and formatting purposes.

### JSON
All Robobot API endpoints return JSON formatted data. POST actions need to be provided JSON formatted request bodies. These bodies should follow a schema as provided by the API reference.

Please note that we require each POST request to include the HTTP header Content-Type with its value set to application/json.

```
curl "https://api.robobot.app/client/entry/submit/<form_id>" \
    -X "POST" \
    -H "Content-Type: application/json"
```

### Authentication
Robobot API has a custom authentication mechanism. It involves including a non-default HTTP header named "ApiCode" with your API requests. Provide your personal ApiCode formatted as xxxx in this header to open up requests to your forms.
Use the "AccessCode" header and populate it with an accesscode to group answer data. We call grouped answers an entry.

An example of a curl request could look like this. Make sure to replace the XXX...X with your obtained access code and provide a form_id. Dive into the API reference to see how you generate your own access code.

```
curl "https://api.robobot.app/client/entry/submit/<form_id>" \
    -X "POST" \
    -H "Content-Type: application/json" \
    -H "ApiCode: xxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxx"
    -H "AccessCode: XXXXXXXXXXXXXXX"
```

## API Reference
This reference can help you explore the Robobot API with your own data.

### Form: Request
Request this endpoint to retrieve the form data for a certain form_id.

`GET /client/form/get/<form_id>`

**Response Class (Status 200):**
```
{
    form: {
        id: 1,
        title: "Mauris mattis",
        background: "#3a87fe",
        buttons: "#ffab01",
        description: "Mauris mattis mi eu suscipit mollis.",
        gaTrackingCode: null,
        image: null,
        allowOpenEntry: true,
        allowAnonymousEntry: true,
        updated: 1581677549
    },
    next: ...
}
```

### Access Code: Request
Request this endpoint to obtain an access code. Use this code as header with your entry submit requests.

1. Enabling "Allow open entries" in your form configuration will allow you to create an entry by providing contact details
2. Enabling "Allow anonymous entries" in your form configuration will allow you to create an entry without providing contact details.

If the email property is missing in the request body and "Allow anonymous entries" is disabled, the API will return a 403.

`POST /client/entry/create/<form_id>`

**Response Class (Status 200):**
```
{
  accessCode: "XXXXXXXXXXXXXXX"
}
```

#### Request Body Example (optional)
```
{
    "firstName": "John",
    "lastName": "Doe",
    "email": "john@example.com"
}
```

### Next: Request
Request this endpoint to retrieve the next element for your form. This endpoint will also return the form data in the form property on the response. If answer data is sent with the request, the API will figure out what element should be next based on your current answer state and the applied order and visibility rules for form elements.

#### Request Body Schema
`POST /client/entry/submit/<form_id>`

```
{
    element_id<string>: [],                     // static-text (submit an empty array for this type)
    element_id<string>: option_id<number>,      // input-options
    element_id<string>: input_text<string>,     // input-text
    element_id<string>: input_number<number>    // input-number
}
```

#### Request Body Example
```
{
    "1": [],                        // static-text
    "2": 2,                         // input-options
    "3": 10,                        // input-number
    "5": ["2"],                     // input-options
    "6": "email@example.com",       // input-text
    "108": 9,                       // input-number
    "153": ["2","3"],               // input-options
    "4": "Nec condimentum ex!"      // input-text
}
```

### Next: Response Types
The response of this request differs based on the current state of your form. When there are no more form elements to return, the API will return the report that is build according to the provided answer data for your entry.

#### Response Type: "static-text"
```
{
    form: ...
    next: {
        id: 122
        title: "Mauris mattis mi eu suscipit mollis!"
        type: "static-text"
        params: {
            body: "Aliquam cursus leo ac fringilla congue. Cras elementum euismod malesuada. Donec mauris dui, commodo ac felis et, blandit efficitur enim."
        }
    },
}
```

#### Response Type: "input-options"
```
{
    form: ...
    next: {
        id: 123
        title: "Ut nec condimentum ex, eget aliquam nulla?"
        type: "input-options"
        description: "Duis vulputate ipsum nec auctor tempus."
        params: {
            options: [
                {
                    id: 0,
                    label: "Praesent",
                    description: "Aenean tincidunt, purus quis."
                },
                {
                    id: 1,
                    label: "Habitasse",
                    description: "Nulla mollis, orci ac pretium tempor"
                },
                {
                    id: 2,
                    label: "Pellentesque",
                    description: "Morbi posuere odio eget"
                },
            ]
        }
    },
    report: ...
}
```

#### Response Type: "input-text"
```
{
    form: ...
    next: {
        id: 124
        title: "Nascetur ridiculus mus"
        type: "input-text"
        description: "Sed tincidunt sem turpis."
        params: {
            placeholder: "Duis luctus orci eget."
        }
    },
}
```

#### Response Type: "input-number"
```
{
    form: ...
    next: {
        id: 125
        title: "Varius posuere is tincidunt sem!"
        type: "input-number"
        description: "Lorem ipsum dolor sit amet."
        params: {
            min: 0
            max: 99
            description: "Augue eu varius posuere."
        }
    },
}
```

### Report
If there are no more elements to view the report will be returned by the submit request. Based on the provided answers the API will return an ordered array of Report Elements.

```
{
    form: ...,
    next: ...,
    report: [
        {
            id: 2,
            type: "html",
            params: {
                body: "<h2>Varius posuere is tincidunt sem!</h2><p>Nulla mollis, orci ac. <a href="https://www.example.com" rel="noopener noreferrer" target="_blank">Augue eu varius</a></p>"
            }
        },
        {
            id: 3,
            type: "score",
            params: {
                score: {
                    points: 20,
                    points_max: 20,
                    score: 100
                }
            }
        },
        {
            id: 4
            type: "button"
            params: {
                label: "Augue eu varius",
                url: "https://www.example.com/",
                color: "#FF0000",
                description: "If eu suscipit mollis."
            }
        }
    ]
}
```

## Local Cache Invalidation
If you want to cache your answers you should think of an invalidation strategy. We update the `Form.updated` will provide a fresh timestamp every time a form is altered. Use this timestamp in a condition to invalidate your local cache.
