# Getting Started

```javascript
const vinegar = require('vinegar')

const background = vinegar.load('background.feature')
const scenario = vinegar.load('example.feature')
const outline = vinegar.load('outline.feature')
```

# Resulting Object with Only a Background

```gherkin
Feature: Set browser
  Background:
    Given I set the browser to "chrome"
```

```json
{
 "backgrounds": [
  {
   "keyword": "Given",
   "text": "I set the browser to \"chrome\""
  }
 ],
 "scenarios": []
}
```

# Resulting Object with Only a Scenario

```gherkin
Feature: Example
  Scenario: I load the example.com homepage.
    Given I start 2 browsers
    When I set the URL to "http://example.org"
    And I set the link to click to "More information..."
    Then the header should read "IANA-managed Reserved Domains".
```

```json
{
 "backgrounds": [],
 "scenarios": [
  [
   {
    "keyword": "Given",
    "text": "I start 2 browsers"
   },
   {
    "keyword": "When",
    "text": "I set the URL to \"http://example.org\""
   },
   {
    "keyword": "When",
    "text": "I set the link to click to \"More information...\""
   },
   {
    "keyword": "Then",
    "text": "the header should read \"IANA-managed Reserved Domains\"."
   }
  ]
 ]
}
```

# Scenario Outlines

```json
{
 "backgrounds": [],
 "scenarios": [
  [
   {
    "keyword": "Given",
    "text": "I start 2 browsers"
   },
   {
    "keyword": "When",
    "text": "I set the URL to \"http://example.org\""
   },
   {
    "keyword": "When",
    "text": "I set the link to click to \"More information...\""
   },
   {
    "keyword": "Then",
    "text": "the header should read \"IANA-managed Reserved Domains\"."
   }
  ],
  [
   {
    "keyword": "Given",
    "text": "I start 2 browsers"
   },
   {
    "keyword": "When",
    "text": "I set the URL to \"http://example.net\""
   },
   {
    "keyword": "When",
    "text": "I set the link to click to \"More information...\""
   },
   {
    "keyword": "Then",
    "text": "the header should read \"IANA-managed Reserved Domains\"."
   }
  ],
  [
   {
    "keyword": "Given",
    "text": "I start 2 browsers"
   },
   {
    "keyword": "When",
    "text": "I set the URL to \"http://example.com\""
   },
   {
    "keyword": "When",
    "text": "I set the link to click to \"More information...\""
   },
   {
    "keyword": "Then",
    "text": "the header should read \"IANA-managed Reserved Domains\"."
   }
  ],
  [
   {
    "keyword": "Given",
    "text": "I start 2 browsers"
   },
   {
    "keyword": "When",
    "text": "I set the URL to \"http://example.edu\""
   },
   {
    "keyword": "When",
    "text": "I set the link to click to \"More information...\""
   },
   {
    "keyword": "Then",
    "text": "the header should read \"IANA-managed Reserved Domains\"."
   }
  ]
 ]
}
```

# Links

- [Burpless](https://github.com/jeromebelleman/vinegar), an asynchronous
  alternative to Cucumber
- [Gherkin Syntax](https://docs.cucumber.io/gherkin)
- [Cucumber Guide](https://docs.cucumber.io/guides)
