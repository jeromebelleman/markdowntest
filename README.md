# Getting Started

```
const vinegar = require('vinegar')

const backgrounds = vinegar.load('background.feature')
const scenarios = vinegar.load('example.feature')
```

# Resulting Object with Only a Background

```gherkin
Feature: Set browser
  Background:
    Given I set the browser to "chrome"
```

```javascript
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

```javascript
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
