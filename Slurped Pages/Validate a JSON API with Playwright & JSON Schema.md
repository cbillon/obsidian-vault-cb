---
link: https://www.mikestreety.co.uk/blog/validate-a-json-api-with-playwright-and-json-schema/
byline: Mike Street
site: Mike Street
date: 2024-03-19T01:00
excerpt: Learn how to validate a JSON API with Playwright
slurped: 2026-08-22T22:17
title: Validate a JSON API with Playwright & JSON Schema
---

2024-03-19T00:00:00.000Z Posted on **19th March 2024**. **5 mins** reading time

Chase Jarvis once said "the best camera is the one that is with you" and I often apply this to other parts of my life. Playwright might not be [the best tool](https://json-schema.org/implementations) to validate JSON with, but if you are already using it and have it set up, then it is better than loading a whole new framework or tool.

This post will walk through calling a JSON API and validating it against a predefined [JSON Schema](https://json-schema.org/). This will flag if the JSON output changes - which might impact users using the API (or your own internal calls to the endpoint).

This post assumes you are already familiar with Playwright and have it set up. The examples will be using **TypeScript**

## Installation

For this test we are going to be using [Ajv](https://www.npmjs.com/package/ajv) - a JSON schema validator.

The following installs it as a dev dependency:

```
npm i ajv -D --save
```

## Generate Schema

The trickiest bit of this is actually generating the schema for the test to check against. There are several tools online - the thing to be careful is to ensure your schema uses the `draft-07` definition - as this is the most recent, stable, release.

For this example, I'll be using the [JSON output](https://alehouse.rocks/api/beers.json) from my [beer blog](https://alehouse.rocks/) - you are more than welcome to use as a sample.

The best tool I've found is by [Itential - JSON to Schema](https://json-to-schema.itential.io/). You paste your JSON output on the left and let it generate the Schema - you can then "Edit" and specify extra properties if desired. Another tool I've used is from [transform](https://transform.tools/json-to-json-schema).

This was the Schema generated for my beers API and I've saved this in my tests folder as `beers.schema.json` - the filename doesn't matter, but it helps to be consistent.

```
{
  "$id": "root",
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "number": {
        "type": "string"
      },
      "title": {
        "type": "string"
      },
      "image": {
        "type": "string"
      },
      "thumbnail": {
        "type": "string"
      },
      "brewery": {
        "type": "string"
      },
      "rating": {
        "type": "string"
      },
      "date": {
        "type": "string"
      },
      "breweries": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "title": {
              "type": "string"
            },
            "slug": {
              "type": "string",
            }
          },
          "required": [
            "title",
            "slug"
          ],
          "additionalProperties": true
        }
      },
      "slug": {
        "type": "string",
      }
    },
    "required": [
      "number",
      "title",
      "image",
      "thumbnail",
      "brewery",
      "rating",
      "date",
      "breweries",
      "slug"
    ],
    "additionalProperties": true
  }
}
```

## Create the Test

The bit you've been waiting for - the test! It might be worth considering setting up a specific `project` for API tests - as you don't need to validate the API schema in many browsers.

The following test is located next to `beers.schema.json` - if the paths differ, then update the path inside the `require`

```
import { test, expect } from '@playwright/test';
import Ajv from 'ajv';

/**
 * @description API JSON Validation
 */
test.describe('JSON Tests', () => {
  // Initialise AJV
  const ajv = new Ajv();

  /**
   * @description Validate Beers JSON
   */
  test('Beers', async ({ request }) => {
	// Load the beer JSON URL
    const response = await (await request.get('https://alehouse.rocks/api/beers.json')).json();

	// Validate the response against the schema file
    const valid = ajv.validate(require('./beers.schema.json'), response);

	// Output the errors text
    if (!valid) {
      console.error('AJV Validation Errors:', ajv.errorsText());
    }

	// If the JSON is valid, the variable is "true"
    expect(valid).toBe(true);
  });
});
```

Hopefully, if you've used the examples above, the test should pass with flying colours. You can verify that it isn't a false positive by changing any of the `string` types to `integer` in the `.schema.json` and Playwright will fail.

The output from AJV isn't the prettiest, but are output both on the command line (starting with `AJV Validation Errors:`) and attached to the report under `stderr`.

## Bonus: Validate a JSON linked from the initial response

It may be your initial JSON request has dynamic links to a separate JSON response you wish to validate - to do this, you can operate an additional `request.get` inside your test.

E.g. If your first JSON API response was like the following:

```
[{
	"link": "https://example.com/api/extra-url-with-date-2024-03-19"
}]
```

You can do the following inside your test:

```
const initialResponse = await (await request.get('https://example.com/api/')).json();

const response = await request.get(initialResponse[0].link);
const responseData = await response.json();
```

You can then validate `responseData` against your schema.