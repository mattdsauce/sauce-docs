---
sidebar_label: Cypress
---

import useBaseUrl from '@docusaurus/useBaseUrl';
import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

# Cypress Integration

:::note Important
Access to this feature is currently limited to Enterprise customers as part of our commitment to providing tailored solutions. We are excited to announce that self-service access is under development and will be released shortly. Stay tuned!
:::

## Introduction

This guide requires an existing Cypress project.<br />
You can alternatively take a look to our [example repository](#example).

Sauce Visual provides an integration with [Cypress](https://cypress.io) through a plugin that you can add to any existing Cypress project.

Sauce Visual introduce a new Cypress command:

- `cy.sauceVisualCheck()`: Takes a screenshot and send it to Sauce Visual for comparison.
- `cy.sauceVisualResults()`: Waits for diff calculations to complete and returns a summary of results.
  See [Test results summary](#test-results-summary) for more details about summary format and sample usage.

:::note Important
Running Cypress tests on Sauce Labs requires SauceCTL to be installed and configured.

[Follow me to learn more](/web-apps/automated-testing/cypress/)
:::

## Quickstart

### Step 1: Add Sauce Visual for Cypress dependency

- Install the Sauce Visual for Cypress plugin in your current project.

```sh
npm install --save-dev @saucelabs/cypress-visual-plugin
```

### Step 2: Configure Cypress to use Sauce Visual for Cypress plugin

- Import the plugin in Cypress project configuration, at the top of the file:

```ts
import { CypressSauceVisual } from '@saucelabs/cypress-visual-plugin';
```

- Register the plugin to Cypress events in Cypress project configuration:

```ts
export default defineConfig({
  e2e: {
    [...]
    setupNodeEvents(on, config) {
      ...
      CypressSauceVisual.register(on, config);
      ...
    },
  },
})
```

- Register Sauce Visual for Cypress commands. Add the following line in your `cypress/support/e2e.ts`:

<Tabs
defaultValue="Cypress 12.17.4+"
  values={[
    {label: 'Cypress 12.17.4+', value: 'Cypress 12.17.4+'},
    {label: 'Cypress 12.17.3 and older', value: 'Cypress 12.17.3'},
  ]}>
<TabItem value="Cypress 12.17.4+">

```ts
import '@saucelabs/cypress-visual-plugin/commands';
```

  </TabItem>
  <TabItem value="Cypress 12.17.3">

```ts
import '@saucelabs/cypress-visual-plugin/build/commands';
```

  </TabItem>
</Tabs>

### Step 3: Add visual tests in your project:

```ts
context('Sauce Demo', () => {
  it('should render correctly', () => {
    ...
    cy.sauceVisualCheck('visual: my-homepage');
    ...
  })
});
```

### Step 4: Configure your Sauce Labs credentials

Sauce Visual relies on environment variables for authentications.<br />
Both `SAUCE_USERNAME` and `SAUCE_ACCESS_KEY` need to be set prior starting your Cypress job.

Username and Access Key can be retrieved from https://app.saucelabs.com/user-settings.

```sh
export SAUCE_USERNAME=__YOUR_SAUCE_USER_NAME__
export SAUCE_ACCESS_KEY=__YOUR_SAUCE_ACCESS_KEY__
```

### Step 5: Run the test

Upon executing your tests for the first time under this step, a visual baseline is automatically created in our system. This baseline serves as the standard for all subsequent Cypress tests. As new tests are run, they are compared to this original baseline, with any deviations highlighted to signal visual changes. These comparisons are integral for detecting any unintended visual modifications early in your development cycle. All test builds, including the initial baseline and subsequent runs, can be monitored and managed through the Sauce Labs platform at [Sauce Visual Builds](https://app.saucelabs.com/visual/builds).

Remember, the baseline is established during the initial run, and any subsequent visual differences detected will be marked for review.

## Advanced usage

### Test results summary

`cy.sauceVisualResults()` returns a summary of test results in format:

```ts
{
    QUEUED: number; // Diffs that are pending for processing. Should be 0 in case the test is completed without any timeouts
    EQUAL: number; // Diffs that have no changes detected
    UNAPPROVED: number; // Diffs that have detected changes and waiting for action
    APPROVED: number; // Diffs that have detected changes and have been approved
    REJECTED: number; // Diffs that have detected changes and have been rejected
}
```

Sample output:

```ts
{ APPROVED: 0, EQUAL: 0, UNAPPROVED: 2, REJECTED: 0, QUEUED: 0 }
```

Sample usage:

```ts
const EXPECTED_TOTAL_UNAPPROVED_DIFFS = 0;

cy.sauceVisualResults().its("UNAPPROVED").should("eq", EXPECTED_TOTAL_UNAPPROVED_DIFFS);
```

### Build name

Sauce Visual for Cypress plugin extends Cypress configuration, allowing to define the context, thus acting on which baselines new snapshots will be compared to. ([More info on baseline matching](/visual-testing/#baseline-matching))

Options:

- `region`: Sauce Labs Region where the new build will be created (default: `us-west-1`)
- `buildName`: Name of the build (default: `Cypress Visual Testing`)
- `project`: Name of the project (default: `None`)
- `branch`: Name of branch (default: `None`)
- `defaultBranch`: Name of the main or default branch (default: `None`)  

They need to be set through the `saucelabs` attribute of `e2e` configuration.

Example:

```javascript
export default defineConfig({
  e2e: {
    [...]
    saucelabs: {
      region: 'eu-central-1',
      buildName: 'SauceDemo - Cypress',
      project: 'Cypress examples',
      branch: 'main',
    },
    [...]
  }
}
```

### Environment variables

Below are the environment variables available for the Sauce Visual for Cypress plugin. Keep in mind that these variables have precedence over the ones defined in Cypress configuration.

| Variable Name           |          | Description                                                                                                                                                                                                                      |
| ----------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `SAUCE_USERNAME`        | required | Your Sauce Labs username. You can get this from the header of app.saucelabs.com                                                                                                                                                  |
| `SAUCE_ACCESS_KEY`      | required | Your Sauce Labs access key. You can get this from the header of app.saucelabs.com                                                                                                                                                |
| `SAUCE_REGION`          |          | The region you'd like to run your Visual tests in. Defaults to `us-west-1` if not supplied. Can be one of the following: <br/> `'eu-central-1'`, `'us-west-1'` or `'us-east-4'`                                                  |
| `SAUCE_VISUAL_BUILD_NAME`      |          | The name you would like to appear in the Sauce Visual dashboard. Defaults to 'Cypress Visual Testing'                                                                                                                                   |
| `SAUCE_VISUAL_BRANCH_NAME`     |          | The branch name you would like to associate this build with. We recommend using your current VCS branch in CI.                                                                                                            |
| `SAUCE_VISUAL_DEFAULT_BRANCH_NAME`     |          | The main branch name you would like to associate this build with. Usually `main` or `master` or alternatively the branch name your current branch was derived from. [Follow me to learn more](../workflows/ci.md) |
| `SAUCE_VISUAL_PROJECT_NAME`    |          | The label / project you would like to associate this build with.                                                                                                                                                                |
| `SAUCE_VISUAL_BUILD_ID`        |          | For advanced users, a user-supplied SauceLabs Visual build ID. Can be used to create builds in advance using the GraphQL API. This can be used to parallelize tests with multiple browsers, shard, or more. <br/> By default, this is not set and we create / finish a build during setup / teardown. |
| `SAUCE_VISUAL_CUSTOM_ID`       |          | For advanced users, a user-supplied custom ID to identify this build. Can be used in CI to identify / check / re-check the status of a single build. Usage suggestions: CI pipeline ID.  |

### Working with custom ID

Linking all your Sauce Visual tests to one build ID can streamline your workflow, especially in CI setups. This enables easy management using the Sauce Visual CLI, like creating builds with custom IDs, adding snapshots, and completing builds. This way, you can efficiently handle multiple tests grouped under the same build in Sauce Visual.

When you use `SAUCE_VISUAL_CUSTOM_ID`, Sauce Visual checks if there's an existing build with that ID. If found, your tests are linked to it; otherwise, Sauce Visual creates a new build. In contrast, using `SAUCE_VISUAL_BUILD_ID` requires an existing build with the provided ID; otherwise, an error occurs.

### Ignored regions

#### Component-based ignored region

Sauce Visual provides a way to ignore a list of components.

An ignored component can be a specific element from the page.

Those ignored components are specified when requesting a new snapshot.

Example:

```javascript
cy.sauceVisualCheck('login-page', {
  ignoredRegions: [
    cy.get('[data-test="username"]'),
  ]
});
```

#### User-specified ignored region

Alternatively, ignored regions can be user-specified areas. A region is defined by four elements.

- `x`, `y`: The location of the top-left corner of the ignored region
- `width`: The width of the region to ignore
- `height`: The heigh of the region to ignore

_Note: all values are pixels_

Example:

```javascript
cy.sauceVisualCheck('login-page', {
  ignoredRegions: [
    {
      x: 240,
      y: 800,
      width: 1520,
      height: 408
    }
  ],
});
```

### Specifying options for Cypress Screenshot

Sauce Visual relies on the native screenshot feature from Cypress. Options in cy.snapshot can be specified in the cy.visualCheck command.

The field `cypress` from `options` will be transmitted as it to `cy.screenshot` command. (See [Cypress documentation](https://docs.cypress.io/api/commands/screenshot#Arguments))

Example:

```javascript
cy.sauceVisualCheck('login-page', {
  cypress: {
    capture: 'viewport',
  }
});
```

### Capturing the DOM snapshot

Sauce Visual does not capture dom snapshot by default. It can be changed in `sauceVisualCheck` options.

Example:
```javascript
cy.sauceVisualCheck('login-page', {
    captureDom: true
});
```

## Limitations

Sauce Visual for Cypress **DOES NOT** support `cypress open`.

Screenshots will be captured and sent to Sauce Labs only when `cypress run` is executed.

## Example

An example project is available [here](https://github.com/saucelabs/visual-examples/tree/main/cypress).
