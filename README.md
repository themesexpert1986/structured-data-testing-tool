# Structured Data Testing Tool

Inspect and test web pages for Structured Data.

Includes both a Command Line Interface for easy ad-hoc testing of URLs and library with extendable API for use when writing tests or building other tools.

## Install

To install the command line tool (`sdtt`), include the `-g` (global) flag when installing:

    npm i structured-data-testing-tool -g

## Features

* Command Line Interface (`sdtt`) and API that can be used with any test framework.
* Accepts a URL, file or string, buffer or stream containing HTML or JSON.
* Automatically detects all Schema.org schemas, in HTML (`microdata`), `JSON-LD` and `RDFa`.
* Can test `<meta>` tags (and custom schemas) for specific tags / fields / values.
* Built-in presets for testing for Twitter, Facebook and Google structured data.
* Support creation of custom presets to test any schema or tests specific to your site.
* Use with a headless browser to test Structured Data injected by client side JavaScript (e.g. Google Tag Manager).

## Usage

### Command Line Interface

```
Usage: sdtt --url <url> [--presets <presets>] [--schemas <schemas]

Options:
  -u, --url      Inspect a URL
  -f, --file     Inspect a file
  -p, --presets  Test for specific markup from a list of presets
  -s, --schemas  Test for a specific schema from a list of schemas
  -i, --info     Show more detailed information about structured data found
  -o, --output   Output test results to a file
  -h, --help     Show help
  -v, --version  Show version number

Usage: sdtt --url <url> [--presets <presets>] [--schemas <schemas]

Examples:
  sdtt --url "https://example.com/article"                Inspect a URL
  sdtt --url <url> --presets SocialMedia                  Test a URL for social media metatags
  sdtt --url <url> --presets Google                       Test a URL for markup inspected by Google
  sdtt --url <url> --presets "Twitter,Facebook"           Test a URL with multiple presets
  sdtt --url <url> -p Twitter -p Facebook                 Test a URL with multiple presets (alternative)
  sdtt --url <url> --schemas Article                      Test a URL for the Article schema
  sdtt --url <url> --schemas "jsonld:Article"             Test a URL for the Article schema in JSON-LD
  sdtt --url <url> --schemas "microdata:Article"          Test a URL for the Article schema in microdata/HTML
  sdtt --url <url> --schemas "rdfa:Article"               Test a URL for the Article schema in RDFa
  sdtt --url <url> --schemas "Article,WPHeader,WPFooter"  Test a URL for multiple schemas
  sdtt --url <url> -s Article -s WPHeader -s WPFooter     Test a URL for multiple schemas (alternative)
  sdtt --url <url> --output results.json                  Output test results to a JSON file
  sdtt --file <path-to-file>.html                         Test file containing HTML
  sdtt --file <path-to-file>.json                         Test file containing JSON-LD
  sdtt --presets                                          List all built-in presets
  sdtt --schemas                                          List all supported schemas
```

Inspect a URL to see what markup is found:

    sdtt --url <url>

Inspect a file to see what markup is found:

    sdtt --file <path to file>

Test a URL contains specific markup:

    sdtt --url <url> --presets "Twitter,Facebook"

Test a URL contains specific schema:

    sdtt --url <url> --schemas "Article"

Test a URL contains specific schema in both JSON-LD and in microdata/HTML:

    sdtt --url <url> --schemas "jsonld:Article,microdata:Article"

Run `sdtt --presets` to list the built-in-presets:

```
NAME                      DESCRIPTION
Google                    Check for common markup used by Google
Twitter                   Suggested metatags for Twitter
Facebook                  Suggested metatags for Facebook
SocialMedia               Suggested markup for integration with social media sites
```

#### Example output from CLI

```
$ sdtt --url https://www.bbc.co.uk/news/world-us-canada-49060410 --presets Google,SocialMedia
Tests

  Schema.org > ReportageNewsArticle - 100% (1 passed, 0 failed)
    ✓  schema in jsonld
    •  @context
    •  @type
    •  url
    •  publisher.@type
    •  publisher.name
    •  publisher.publishingPrinciples
    •  publisher.logo.@type
    •  publisher.logo.url
    •  datePublished
    •  dateModified
    •  headline
    •  image.@type
    •  image.width
    •  image.height
    •  image.url
    •  thumbnailUrl
    •  author.@type
    •  author.name
    •  author.logo.@type
    •  author.logo.url
    •  author.noBylinesPolicy
    •  mainEntityOfPage
    •  video.@list[0].@type
    •  video.@list[0].name
    •  video.@list[0].description
    •  video.@list[0].duration
    •  video.@list[0].thumbnailUrl
    •  video.@list[0].uploadDate
    •  video.@list[1].@type
    •  video.@list[1].name
    •  video.@list[1].description
    •  video.@list[1].duration
    •  video.@list[1].thumbnailUrl
    •  video.@list[1].uploadDate

  Google > ReportageNewsArticle > #0 (jsonld) - 100% (12 passed, 0 failed)
    ✓  ReportageNewsArticle
    ✓  @type
    ✓  author
    ✓  datePublished
    ✓  headline
    ✓  image
    ✓  publisher.@type
    ✓  publisher.name
    ✓  publisher.logo
    ✓  publisher.logo.url
    ✓  dateModified
    ✓  mainEntityOfPage

  Facebook - 100% (8 passed, 0 failed)
    ✓  must have page title
    ✓  must have page type
    ✓  must have url
    ✓  must have image url
    ✓  must have image alt text
    ✓  should have page description
    ✓  should have account username
    ✓  should have locale

  Twitter - 100% (7 passed, 0 failed)
    ✓  must have card type
    ✓  must have title
    ✓  must have description
    ✓  must have image url
    ✓  must have image alt text
    ✓  should have account username
    ✓  should have username of content creator

Statistics

  Number of Metatags: 38
  Schemas in JSON-LD: 1
     Schemas in HTML: 0
      Schema in RDFa: 0
  Schema.org schemas: ReportageNewsArticle
       Other schemas: 0
     Test groups run: 5
  Optional tests run: 71
 Pass/Fail tests run: 28

Results

    Passed: 28 	(100%)
  Warnings: 0 	(0%)
    Failed: 0 	(0%)

  ✓ 28 tests passed with 0 warnings.

Use the option '-i' to display additional detail.
```

### API

#### How to test a URL

You can integrate Structured Data Testing Tool with a CD/CI pipeline by using the API.

```javascript
const { structuredDataTest } = require('structured-data-testing-tool')
const { Google, Twitter, Facebook } = require('structured-data-testing-tool/presets')

const url = 'https://www.bbc.co.uk/news/world-us-canada-49060410'
 
let result

structuredDataTest(url, { 
  // Check for compliance with Google, Twitter and Facebook recommendations
  presets: [ Google, Twitter, Facebook ],
  // Check the page includes a specific Schema (see https://schema.org/docs/full.html for a list)
  schemas: [ 'ReportageNewsArticle' ]
})
.then(res => {
  console.log('✅ All tests passed!')
  result = res
})
.catch(err => {
  if (err.type === 'VALIDATION_FAILED') {
    console.log('❌ Some tests failed.')
    result = err.res
  } else {
    console.log(err) // Handle other errors here (e.g. an error fetching a URL)
  }
})
.finally(() => {
  if (result) {
    console.log(
      `Passed: ${result.passed.length},`,
      `Failed: ${result.failed.length},`,
      `Warnings: ${result.warnings.length}`,
    )
    console.log(`Schemas found: ${result.schemas.join(',')}`)

    // Loop over validation errors
    if (result.failed.length > 0)
      console.log("⚠️  Errors:\n", result.failed.map(test => test))
  }
})
```

#### How to test a local HTML file

You can also test HTML in a file by passing it as a string, a stream or a readable buffer.

```javascript
const html = fs.readFileSync('./example.html')
structuredDataTest(html)
.then(response => { /* … */ })
.catch(err => { /* … */ })
```

### How to define your own tests

The built-in presets only cover some use cases and are only able to check if values are defined (not what they contain).

With the API you can use [JMESPath query syntax](http://jmespath.org) to define your own tests to check for additional properties and specific values. You can mix and match tests with presets.

```javascript
const testUrl = 'https://www.bbc.co.uk/news/world-us-canada-49060410'

const options = {
  tests: [
    // Check 'NewsArticle' schema exists in JSON-LD
    { test: 'NewsArticle', expect: true, type: 'jsonld' },

    // Check a 'NewsArticle' schema exists with 'url' property set to the value of the  variable 'url'
    { test: 'NewsArticle[*].url', expect: testUrl },

    // A similar check as above, but won't fail (only warn) if the test doesn't pass
    { test: 'NewsArticle[*].mainEntityOfPage', expect: testUrl, warning: true },

    // Test for a Twitter meta tag with specific value
    { test: '"twitter:domain"' expect: 'www.bbc.co.uk', type: 'metatag' }
  ]
}

structuredDataTest(testUrl, options)
.then(response => { /* … */ })
.catch(err => { /* … */ })
```

### How to define your own presets

A preset is a collection of tests.

There are built-in presets you can use, you can list them with `--presets` option using the CLI. You can also easily define your own custom presets when using the API. The Command Line Interface only supports built-in presets.

Presets must have a `name` (which should ideally be unique, but does not have to be) and `description` and an array of `test` objects in `tests`. Both `name` and `description` be arbitrary strings, `tests` should be an array of valid `test` objects.

You can optionally group tests by specifying a value for `group` and set a default schema to use for all tests in `schema`. These can be arbitrary strings, though it's recommended schemas reflect Schema.org schema names.

If a test explicitly defines it's own `group` or `schema`, that will override the default value for the preset for that specific test (which may impact how results are grouped).

Presets can contain other presets using the `presets` property (an array).

Presets can have `conditional` property, which contains a `test` object, in which case the tests in the preset will only only be run if the conditional test passes.

#### Preset Example 1

```javascript
const url = 'https://www.bbc.co.uk/news/world-us-canada-49060410'

// This test shows how you can use different types of tests in one preset.
const MyCustomPreset = {
  name: 'My Custom Preset', // Required
  description: 'Test ReportageNewsArticle JSON-LD data is defined and twitter metadata was found', // Required
  tests: [ // Required (unless 'presets' is specified)
    { test: 'ReportageNewsArticle', type: 'jsonld' },
    { test: '"twitter:card"', type: 'metatag' },
    { test: '"twitter:domain"', expect: 'www.bbc.co.uk', type: 'metatag', }
  ],
  // Additional options you can use in a preset:
  // group: 'My Group Name', // A group name can be used to group tests in a preset (defaults to preset name)
  // schema: 'NewsArticle', // Specify a schema at the top level if all the tests in the preset apply to the same schema
  // presets: [] // A preset can also invoke other presets, making it easy to re-use custom tests
  // conditional: {} // Define a conditional `test`, which is evaluated to determine if the preset should run
}

const options = {
  // The 'presets' argument should be an array of preset objects
  presets: [ MyCustomPreset ],
  // If you just want to detect a schema exists, you can populate the
  // the 'schemas' option with a list of schema names (as strings).
  schemas: [ 'ReportageNewsArticle' ],
  // By default, any structured data detected will automatically be tested.
  // Set 'auto' to 'false' if you want to disable this (defaults to 'true').
  // This may mean you miss some errors, but make make debugging easier.
  auto: false
}

structuredDataTest(url, options)
.then(response => { /* … */ })
.catch(err => { /* … */ })
```

#### Preset Example 2

This is the code for one of the built-in presets, it tests for the **ClaimReview** schema.

It shows how to write a preset that will automatically run against all instances of a given schema found.

This is useful to be able to do when you have multiple instances of the same schema on page.

NB: This example is quite simple and doesn't try and validate the contents of the properties in the schema or check for invalid properties on the schema.

```javascript
const ClaimReview = {
  name: 'ClaimReview',
  description: 'A fact-checking review of claims made (or reported) in some creative work (referenced via itemReviewed).',
  // If you add 'schema' property to a preset **and** write tests that start with a selector like `ClaimReview[*]`
  // (i.e. with the schema name followed by an asterisk in the selector) then those tests will automatically
  // be run against every instance of that schema found, so you can easily find where an error is if there are
  // multiple instances of the same schema on a page.
  schema: 'ClaimReview',
  // A 'conditional' on a preset or test is just a normal test object. If it fails to pass, the tests in the
  // preset (or the individual test, if it is used on a test) will not be run.
  conditional: {
    test: 'ClaimReview'
  },
  tests: [
    // Expected by Google
    { test: `ClaimReview` },
    { test: `ClaimReview[*]."@type"`, expect: 'ClaimReview' },
    { test: `ClaimReview[*].url` },
    { test: `ClaimReview[*].reviewRating` },
    { test: `ClaimReview[*].claimReviewed` },
    // Warnings
    { test: `ClaimReview[*].author`, warning: true },
    { test: `ClaimReview[*].datePublished`, warning: true },
    { test: `ClaimReview[*].itemReviewed`, warning: true },
  ],
}

module.exports = {
  ClaimReview
}
```

### Test options

#### test
```
Type: string
Required: true
```

The value for `test` should be a valid [JMESPath query](http://jmespath.org).

Examples of JMESPath queries:

`Article`
Test `Article` schema found.

`Article[*].url`
Test `url` property of any `Article` schema found.

`Article[0].headline`
Test `headline` property of first `Article` schema found.

`Article[1].headline`
Test `headline` property of second `Article` schema found.

`Article[*].publisher.name`
Test `name` value of `publisher` on any `Article` schema found.

`Article[*].publisher."@type"`
Test `@type` value of `publisher` on any `Article` schema found.

`"twitter:image" || "twitter:image:src"`
Check for a metatag named either `twitter:image` -or- `twitter:image:src`

Tips:

* Use double quotes to escape special characters in property names.
* You can `console.log()` the `structuredData` property of the response object from `structuredDataTest()` to see what sort of meta tags and structured data was found to help with writing your own tests.

#### type
```
Type: string ('json'|'rdfa'|'microdata'|'any')
Required: false
Default: 'any'
```

You can specify a `type` to indicate if markup should be in `jsonld`, `rdfa` or `microdata` (HTML) format.

You can also specify a value of `metatag` to check `<meta>` tags.

If you do not specify a type for a test, a default of `any` will be assumed and all types will be checked (and if any source matches, the test will pass).

If you specifically want to test for a value and you know if it is JSON-LD, RDFa or microdata you should specify the explicit type for the test to check.

#### expect
```
Type: boolean|string|RexExp
Required: false
Default: true
```

You can specify a value for `expect` that is either a boolean, a string or a Regular Expression object (defaults to `true`).

* A value of `true` indicates the property must exist (but does not check it's value).
* A value of `false` that indicates the value must not exist.
* A Regular Expression is evaluated against the test query (the test passes if a test for expression passes).
* Any other value is treated as a string and the value of the property should exactly match it.

When using a Regular Expression if the query points to an array then the test will pass if any item in the array matches the Regular Expression.

Examples of how to use [Regular Expressions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp) with the `expect` option:

* `expect: /^[0-9]+$/g` // Value being tested should only contain numbers
* `expect: /^[A-z]+$/g` // Value being tested should only contain letters
* `expect: /^[A-z0-9 ]+$/g` //  Value should only contain letters, numbers and spaces

You can use regular expressions to validate dates, specific values, URLs, etc.

#### warning
```
Type: boolean
Required: false
Default: false
```

When `warning` is set to `true`, if the test does not pass it will only result in a warning.

The default is `false`, meaning if the test fails it will be counted as a failure.

#### optional
```
Type: boolean
Required: false
Default: false
```

When the `optional` property is set to `true` on a test, a test will not count as either passed or failed, but the test will still be run and the result able to be inspected.

Optional tests do not count towards the total number of tests run, test passed or tests failed. They will show up in results in the Command Line Interface if they pass, but not if they fail; however passing optional tests appear differently to other tests in the results to make it clear they are optional checks. 

You can use `--info/-i` on the CLI or inspect the `optional` property on the response from the API to see the result of any test that has `optional` property set on it. However, if an optional test fails because the property it was testing does not exist, it will not be displayed in the CLI. If a property is optional but recommended, use the `warning` option instead.

Note: Strictly speaking, in principle no specific properties on Schema.org objects are "required" but in practice implementations by vendors like Google have some "required" or expected properties and also respect some "optional" properties; this option is useful for writing tests that don't fail if a valid, but not necessarily required, property is not found.

The default is `false`.

#### conditional
```
Type: object
Required: false
Default: undefined
```

A `conditional` object can contain a conditional test to be run, to determine if the test itself should be run.

If the conditional test fails, the test will not be run (and it will not be included in the test results). If the conditional test passes, the test will be run as it otherwise would be if the condition wasn't specified.

This is considered advanced usage, to help avoid having to write overly complex test statements. Conditional test objects use the same syntax as regular test objects, but conditional tests are not included in the results.

It is particularly useful for checking if it is appropriate to run a group of tests. For example, it is used by internal presets to check if a schema exists; if it does then all the tests for that schema are run (and required tests must pass), but if a schema does not exist then none of the tests for that schema are run.

#### group
```
Type: string
Required: false
Default: undefined
```

You can pass a string for the `group` value to indicate how tests should be grouped when displaying results. You do not need to specify a group if tests are in a preset, by default the preset name will be used.

#### groups
```
Type: array of strings
Required: false
Default: undefined
```

You can pass an array of strings to be used to group tests. This used internally to group tests by the structured data testing tool and is considered advanced usage for edge case situations like creating tests dynamically.

#### schema
```
Type: string
Required: false
Default: undefined
```

You can pass a `schema` value that indicates what schema a test is for. Tests in different presets can test the same schema, tests in the same preset can also test multiple schemas.

This is intended as an option to control how tests are grouped when displaying results, the value is not checked for validity and is considered advanced usage for edge case situations.

### Testing with client side rendering

If a page uses JavaScript with client side rendering to generate Structured Data, you can use a tool like [Puppeteer](https://github.com/GoogleChrome/puppeteer) (a headless Chrome API) to fetch the HTML and allow any client side JavaScript to run and then test the rendered page with the Structured Data Testing Tool.

This can be used to test pages that rely on client side injection with tools like Google Tag Manager to add Structured Data to pages.

Notes:

* Puppeteer is a large package (~272 MB) and must be installed separately.
* You can only use Puppeteer with the API, not the Command Line Interface.

Example of how to use `puppeteer` with `structured-data-testing-tool` to write a test that relies on client side JavaScript:

```javascript
const { structuredDataTest } = require('structured-data-testing-tool')
const puppeteer = require('puppeteer');

(async () => {
  const url = 'https://www.bbc.co.uk/news/world-us-canada-49060410'
  
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto(url, { waitUntil: 'networkidle2' });
  const html = await page.evaluate(() => document.body.innerHTML);
  await browser.close();
  
  await structuredDataTest(html)
  .then(response => { console.log("All tests passed.") })
  .catch(err => { console.log("Some tests failed.") })
})();
```

### Contributing

Contributions are welcome - especially additions and improvements to the built-in presets.

This can include bug reports, feature requests, ideas, pull requests, examples of how you have used this tool (etc).

Please see the [Code of Conduct](https://github.com/glitchdigital/structured-data-testing-tool/blob/master/CODE_OF_CONDUCT.md) and complete the issue and/or Pull Request templates when reporting bugs, requesting enhancements or contributing code.

Feedback and insight on how you use Structured Data Testing Tool is also very helpful.
