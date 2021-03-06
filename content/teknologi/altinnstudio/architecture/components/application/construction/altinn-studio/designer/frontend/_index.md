---
title: Application construction components - Designer Front-end
linktitle: Front-end
description: Description of the application construction components for Altinn Studio Designer Front-end
toc: true
---

## React architecture

{{%panel%}}
**NOTE:** Parts of the front-end is currently built in .NET Core.
This will gradually be ported over to React as we work with the different functional areas.
{{%/panel%}}

The front-end of Altinn Studio is build using React and Redux. Each functional area has its own React application, complete with Redux store/reducer when needed. 

Navigation/administration of the different applications is done from top-level applications, which import the applications for the different functional areas. 

The React front-end for Altinn Studio is split into two top level applications:

- dashboard
- service-development

In addition to these top level application, each feature/functional area will have its own 
React application which will be imported to the relevant top level application as a _subapp_ (see https://redux.js.org/recipes/isolatingsubapps ).

{{%panel%}}
**Remember:** New subapps must be configured in the top level application's Dockerfile and in the Designer's gulp-file.  
This is not necessary for shared components.
{{%/panel%}}

In addition to feature applications there is also a component library of shared components, that can be reused by all the applications. An example is navigation components. 

See below diagram for an overview of the different applications.

{{%panel%}}
**NOTE:** Runtime applications are part of Altinn Studio Apps, and will not be described here.
{{%/panel%}}

![React app architecture](react-app-architecture.svg "React app architecture")

## service-development
This is a top-level application, and will only handle simple operations like navigation to the different subapps. It will not have access to the store of any of the subapps.

### Header and Navigation
Material UI (applicatiopn bar and drawer) components are customized with altinn studio styles for the header and navigation in Altinn Studio.
A third-party library, [React Routing](https://reacttraining.com/react-router/web/guides/quick-start), is used together with Material UI to handle navigation.
When the user clicks on a header/side navigation link, the route changes and the subapp specific to the route is rendered.

### Header Menu (Application bar)
Application bar component is the Altinn Studio's header menu(navigation links at the top). React router library is used to handle those navigations.
Header menu has different user interface on desktop and tablet.
The display text and the links for navigation are built as object in a configuration file
[appbarconfig](https://github.com/Altinn/altinn-studio/blob/master/src/react-apps/applications/shared/src/navigation/main-header/appBarConfig.tsx).

The configuration object in the file is iterated and the application bar is rendered.
The styles specific to the component are placed inside
[the component file](https://github.com/Altinn/altinn-studio/blob/master/src/react-apps/applications/shared/src/navigation/main-header/appBar.tsx).
In addition to the navigation menu, a breadcrumb is also rendered in tablet view.

### Side Menu (Drawer Menu)
Drawer menu component is the Altinn Studio's side menu which can be found on the left.
It displays a list of Icons by default and on hover expands the menu and lists text by the side of the icon.
It will render a list of navigation links based on the selected header menu.

Side menu has different user interfaces in desktop and tablet. In tablet, only text is displayed and it slides in from left when "Menu" button is clicked.

Two different components are created to acheive this:

- [LeftDrawerMenu](https://github.com/Altinn/altinn-studio/blob/master/src/react-apps/applications/shared/src/navigation/drawer/LeftDrawerMenu.tsx)
- [TabletDrawerMenu](https://github.com/Altinn/altinn-studio/blob/master/src/react-apps/applications/shared/src/navigation/drawer/TabletDrawerMenu.tsx)

The styles specific to the side menu is added in a separate
[style file](https://github.com/Altinn/altinn-studio/blob/master/src/react-apps/applications/shared/src/navigation/drawer/leftDrawerMenuStyles.ts).
Similar to the header menu, the side menu is also rendered by looping over the menu settings object which is available in a separate
[configuration file](https://github.com/Altinn/altinn-studio/blob/master/src/react-apps/applications/shared/src/navigation/drawer/drawerMenuSettings.ts)

## service-overview
Implementation not started. Details will be made available once this application is created.

## ux-editor
The general concept is that there is a JSON file (_FormLayout.json_) where the components that are to be part of a form are specified.
This includes the component types, texts, order, etc. This file is then parsed to display the form.

The _ux-editor_ application is used to create/change this file.
The components specified in the file are rendered to visually display the result. 

### The components

All the components that can be added in the ux-editor are React components, and when they are added, the _FormLayout_-file is updated and the component is rendered.

Currently, available components are:

- HeaderComponent
- InputComponent
- CheckboxContainerComponent
- TextAreaComponent
- RadioButtonContainerComponent
- DropdownComponent
- FileUploadComponent
- ThirdPartyComponent (imported from outside Altinn Studio)

Each component has a defined set of props that it expects as input. It's up to the parent component to provide these.
In addition, props can be mapped directly from the Redux store. 

When an end user makes changes in a form (for example type something in a text box), an event is triggered which triggers an action, handled by a _dispatcher_. 

```typescript
/**
 * This is the event handler that triggers the Redux Actions
 * that is sendt to the different Action dispatcher.
 * This event handler is used for all form components rendered from this
 */
public handleComponentDataUpdate = (callbackValue: any): void => {
  if (!this.props.component.dataModelBinding) {
    return;
  }

  FormFillerActionDispatchers.updateFormData(
    this.props.id,
    callbackValue,
    this.props.dataModelElement,
  );
  ExternalApiActionDispatchers.checkIfApiShouldFetch(this.props.id, this.props.dataModelElement, callbackValue);
  RuleConnectionActionDispatchers.checkIfRuleShouldRun(this.props.id, this.props.dataModelElement, callbackValue);
}
```

### Containers

Components are rendered within containers.
There is a base container which is always available, and unless otherwise specified, components are rendered within the base container.
Any other containers that are defined in FormLayout.json are also rendered inside the base container.
When an app developer first creates a form, the base container is automatically generated with the first component added. 

An app developer can add new containers to group together fields in a form.
These groups may be _repeating_ if the data model allows for this.
If a group is defined as repeating, it must be connected to the relevant repeating group in the data model. 

### Redux

Redux is used to manage the states of the ux-editor.

#### AppConfigState

Which mode is the application in.

```typescript
export interface IAppConfigState {
  designMode: boolean;
}
```

#### DataModelState

Information about the data model elements. Based on JSON file generated from XSD data model.

```typescript
export interface IDataModelState {
  model: IDataModelFieldElement[];
  fetching: boolean;
  fetched: boolean;
  error: Error;
}
```

#### RuleModelState

Information about the rules defined for the app.

```typescript
export interface IRuleModelState {
    model: IRuleModelFieldElement[];
    fetching: boolean;
    fetched: boolean;
    error: Error;
  }
```

#### TextResourceState

All text resources for the app.

```typescript
export interface ITextResourcesState {
  resources: ITextResource[];
  language: string;
  fetching: boolean;
  fetched: boolean;
  error: Error;
}
```

#### FormFillerState

All form data and any validation errors on this form data.

```typescript
export interface IFormFillerState {
  formData: any;
  validationErrors: any;
}
```

### Form data format

The form data is stored in the state as _key-value pairs_ with data model element as the key.
For example, a field connected to `melding.adresse.postnummer` in the data model will be stored as:

```typescript
formData: {
	melding.adresse.postnummer : "1234"
}
```

If a field is inside a repeating group, an index will be added in the key to specify which instance of the group the data belongs to.
For example, if the group `melding.adresse` is defined as repeating and the end user has added 3 instances of this group, it would result in the following 
form data being stored.

```typescript
formData: {
	melding.adresse[0].postnummer : "1234",
	melding.adresse[1].postnummer : "2345",
	melding.adresse[2].postnummer : "4567"
}
```

### Reducer
Redux reducers are used to update the different states in the store. There is one reduer per state.
The reducers listen to the actions that are dispatched when changes are made.

### Action types
Action types are type definitions for events that trigger an update of the store. For example:

```typescript
// All update form data actions
export const UPDATE_FORM_DATA: string = `${moduleName}.UPDATE_FORM_DATA`;
export const UPDATE_FORM_DATA_FULFILLED: string = `${moduleName}.UPDATE_FORM_DATA_FULFILLED`;
export const UPDATE_FORM_DATA_REJECTED: string = `${moduleName}.UPDATE_FORM_DATA_REJECTED`;

```

### Actions
Actions are the events that are triggered when a change is made.

An action contains the action type, and any metadata needed to complete the action. For example:

```typescript
export interface IUpdateFormDataAction extends Action {
  formData: any;
  componentID: string;
  dataModelElement: IDataModelFieldElement;
}
```

Action creators create the actions, based on the interfaces defined for the action. For example:

```typescript
export function updateFormDataAction(
  componentID: string,
  formData: any,
  dataModelElement: IDataModelFieldElement,

): IUpdateFormDataAction {
  return {
    type: ActionTypes.UPDATE_FORM_DATA,
    formData,
    componentID,
    dataModelElement
  };
}
```

The actions are _dispatched_ by an action dispatcher.

### Redux saga
Redux saga is the middleware used to process information before the store is updated.
All logic used in a saga should be an exported function in the `utils`-folder.
This is decided since we need to split up the logic from the fetching of data, so we have a more testable codebase.
The saga only fetches data from the state, and sends the appropriate data to utils-functions.

An example is asyncronous calls to backend APIs to get data, or submit data.

Each saga defines methods that complete different tasks, connected to actions.
These methods are called via listeners that listen to the actions that are being dispatched.
There are different sagas for all the different functional areas.

```typescript
/**
 * Define the saga for the UPDATE_FORM_DATA event
 */
function* updateFormDataSaga(action: ActionType) {
  try {
    const relevantData = yield selectRelevantStateObjects(...);
    ...
    doRelevantLogic(relevantData);
    ...
    yield call(updateFormDataSagaFulfilled, ... );
  } catch (err) {
    yield call(updateFormDataSagaRejected, err);
  }
}

/**
 * Define a listener for the UPDATE_FORM_DATA event
 */
export function* watchUpdateFormDataSaga(): SagaIterator {
  yield takeLatest(FormFillerActionTypes.UPDATE_FORM_DATA, updateFormDataSaga);
}
```

## datamodel
Implementation not started. Details will be made available once this application is created.

## logic-rules
Implementation not started. Details will be made available once this application is created.

## workflow
Implementation not started. Details will be made available once this application is created.

## translations
Implementation not started. Details will be made available once this application is created.

## autosave
If you make changes to the form, the app should auto-save. This is done by triggering a new save-action after the addition, update or delete of:

- components
- containers
- api-connections
- dynamic rules
- conditonal rendering rules

The auto-save also triggers when you update the order of the components inside the form.
