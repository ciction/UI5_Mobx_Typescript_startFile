## Application Details
|               |
| ------------- |
|**Generation Date and Time**<br>Fri Dec 05 2025 12:54:22 GMT+0000 (Coordinated Universal Time)|
|**App Generator**<br>SAP Fiori Application Generator|
|**App Generator Version**<br>1.19.6|
|**Generation Platform**<br>SAP Business Application Studio|
|**Template Used**<br>Basic|
|**Service Type**<br>None|
|**Service URL**<br>N/A|
|**Module Name**<br>mobx_ts|
|**Application Title**<br>Mobx Ts|
|**Namespace**<br>be.cds|
|**UI5 Theme**<br>sap_horizon|
|**UI5 Version**<br>1.143.0|
|**Enable Code Assist Libraries**<br>False|
|**Enable TypeScript**<br>True|
|**Add Eslint configuration**<br>False|

## mobx_ts

An SAP Fiori application.

### Starting the generated app

-   This app has been generated using the SAP Fiori tools - App Generator, as part of the SAP Fiori tools suite.  To launch the generated application, run the following from the generated application root folder:

```
    npm start
```

#### Pre-requisites:

1. Active NodeJS LTS (Long Term Support) version and associated supported NPM version.  (See https://nodejs.org)




---

## MobX Integration Summary

We have integrated **MobX** into the UI5 TypeScript project to manage application state more efficiently. This replaces the traditional approach of manually manipulating `JSONModels` and extensive Controller logic with a reactive, store-based architecture.

### 1. Dependencies & Tooling Configuration
To allow standard NPM packages (like MobX) to work within the UI5 AMD loader environment, we updated the project configuration.

* **Dependencies (`package.json`):**
    * Added `mobx`: The core state management library.
    * Added `ui5-mobx`: A wrapper library that bridges MobX observables to a UI5 Model implementation.
    * Added `ui5-tooling-modules`: A build task/middleware that converts CommonJS/ESM Node modules into UI5-compatible modules on the fly.

* **UI5 Configuration (`ui5.yaml`):**
    * Registered `ui5-tooling-modules-task` and `middleware`.
    * Added `cpro.js.ui5.mobx` to `includeDependency` to ensure the adapter library is correctly bundled during the build.

* **TypeScript Configuration (`tsconfig.json`):**
    * Added a path mapping for `cpro/js/ui5/mobx/*`. This ensures TypeScript can find the types for the `ui5-mobx` library, which relies on a specific UI5 namespace rather than standard module resolution.

### 2. The Store (`webapp/store/MainStore.ts`)
We moved the business logic and state out of the Controller and into a dedicated Store class.

* **Observables:** Properties like `mainSwitch` and the switch states for Scenarios A and B are marked as observables. Changes to these automatically trigger updates.
* **Computed Values:** Complex logic (e.g., `isTrafficLightOn`, `isScenarioA_ConditionMet`) is implemented as getters. MobX automatically recalculates these only when their dependencies change. This removes the need for manual event handlers to check conditions.
* **makeAutoObservable:** Used in the constructor to automatically detect and register properties and methods, simplifying the boilerplate.

### 3. Controller Integration (`webapp/controller/View1.controller.ts`)
The Controller has been significantly slimmed down.

* **Initialization:** Instead of creating a `JSONModel`, we instantiate the `MainStore`.
* **MobxModel:** We wrap the store in a `MobxModel` (provided by `ui5-mobx`) and set it as the View's model. This model acts as a proxy, translating UI5 data binding requests into MobX observable lookups.

### 4. View Binding (`webapp/view/View1.view.xml`)
The XML View utilizes standard UI5 binding syntax, but it is now powered by reactivity.

* **Two-Way Binding:** Controls like `<Switch>` are bound directly to store properties (e.g., `{/scenarioA_Switches/s1}`). Toggling the UI updates the store immediately.
* **Computed Binding:** Properties like `visible` or `status` are bound to Computed getters (e.g., `{/isTrafficLightOn}`).
    * *Benefit:* We no longer need formatters or controller functions to toggle visibility. If the state changes in the store, the View updates automatically.

### 5. Deployment & Launch Updates
* **Launch Configuration:** Updated `.vscode/launch.json` to use `npx` for running the Fiori tools. This ensures the project uses the local project dependencies rather than global installations, improving consistency across different development environments.