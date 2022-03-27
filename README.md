# Architecture

The application is divided into several layers according to the clear architecture in order to ensure maintainability and scalability.

The main advantage of the architecture is that if we need to replace libraries (e.g. Mobx with React) we will change only one module. The rest of the application will not be affected.

Another advantage is that we can just copy the domain and the application layer to any application written in JS/TS, because the layers don't have any dependencies.

```txt
 ______         ____            ________________         _____________ 
| User |------>| UI |<-------->| Infrastructure |<----->| Application |
''''''''       ''''''          | Axios, MOBX ...|       '''''''|'''''''
                               ''''''''''''''''''              |
                                                         ______|______
                                                        |   Domain    |
                                                        '''''''''''''''
```

## Domain layer

The layer contains entity definition and has no dependencies.

Each entity and relationships between entities are described by types and interfaces.
The layer has basic pure functions for working with entities and describing the general rules of the application.

## Application layer

The layer of use cases. It contains functions that describe the behavior of the system.

For example there's an function which describes use case when the user edits the company. It's just a sequence:
 * At the beginning we get the whole info about the company from the server.
 * Then we modify data and check it using validation.
 * If there are errors we use notification service to show them.
 * Otherwise, if everything is ok we save new info to the application store and send the information to the server.

```ts
interface ApiService {
  company: {
    load: () => Promise<Company>;
    save: (company: Company) => Promise<Company>;
  }
}
interface StoreService { ... }
interface ValidationService { ... }
interface NotificationService { ... }

const editCompany = async (form: CompanyForm) => {
  const company = await apiService.company.load();
  const newInfo = { ...company, ...form };
  const errors = validationService.validCompany(newInfo);
  if (errors) {
    notificationService.companyValidationError(errors);
  } else {
    storeService.company.save(newInfo);
    await apiService.company.save(newInfo);
  }
}
```
THE MOST IMPORTANT THING IS THAT: any use case knows nothing about what apiService and validationService are.
The layer knows nothing about libraries or frameworks used to implement the services.
The layer only knows about interfaces of apiService and validationService.
The both of them must be implemented in the next layer according the intrefaces.
It's true for any service in the function.

Any interface of the layer is a port. It shows what the layer needs to be implemented in the next layer

## Infrastructure layer

This one contains all services implemented in accordance with the interfaces from the application layer.
We can use any libraries here like Axios, Mobx and etc.

### Adapters

Adapters can be thought of as a layer that transforms data passed from one layer to another. It's mostly used to connect the infrastructure layer and react application.
For example there can be some hooks which connects react components with the mobx state or some services created in the infrastructure layer.

## Modules

The application also can be divided into modules. Each module must implement the architecture described above. Any module has to be separated from other modules, any imports have to be avoid if it's possible.

# Style guide

## Core rules

1. Use meaningful names for one-line functions, like `map`,`filter`, `reduce`.
1. Use aliases for import path.
1. Use named imports/exports.
1. `any` is prohibited to use. 
1. Define types for data retrieved from server.
1. Use `@todo` and `@hack` comments for future improvements and fixes.
1. Always use `eslint` and `prettier`.

## Component guidelines

1. Keep components small and functional.
1. Use [classnames](https://www.npmjs.com/package/classnames) package for defining multiple css classes.
1. Always use named export, avoid `export default`.
1. Use `UpperCamelCase` for files containing components.
1. Use `.tsx` extension for components

### Functional component template

```tsx
// Defines props types
export interface ComponentProps {
  a: number;
  b?: string;
}

export const Component: React.FC<ComponentProps> = ({ a, b = '' }) => { ... };
```

```tsx
export const Component: React.FC<ComponentProps> = React.memo(({ a, b = '' }) => { ... });
```

To avoid the case the name of the component is lost in React Dev Tools when it's wrapped `observer` function:

```tsx
export const Component: React.FC<ComponentProps> = observer(function Component({ a, b = '' }) { ... });
```

### Component folder structure 

`/SomeComponent/` folder contains:

- `index.ts` - should re-export needed components and variables (works like facade pattern)
- `types.ts` - contains types and interfaces used for component and it's child components (which should be placed in the same folder).
- `functions.ts` - the same thing, but for functions.
- `SomeComponent.module.scss` - component styles.
- `SomeComponent.tsx` - component itself.
- `SomeComponent.test.ts` - file for component specs _(if necessary)_
- `/AnotherComponent` - folder for child component, used only as part of `SomeComponent`

#### Component example

`index.ts` file:

```tsx
export * from './SomeComponent';
export { countPoints } from './functions';
export type { User } from './types';
```

`types.ts` file:

```tsx
export type ButtonType = 'primary' | 'outlined' | 'error';

export type User = {
  id: number;
  name: string;
};
```

`SomeComponent.tsx` file:

```tsx
import React, { useState } from 'react';
import cn from 'classnames';
import { AnotherComponent } from './AnotherComponent';
import { ButtonType, User } from './types';
import s from './SomeComponent.module.scss';

interface SomeComponentProps {
  users: User[];
  onSelectUser: (userId: number) => void;
  onDelete: () => void;
  type?: ButtonType;
  className?: string;
  disabled?: boolean;
}

export const SomeComponent: React.FC<SomeComponentProps> = (props) => {
  const { children, users, onSelectUser, className, type = 'primary', onDelete, disabled } = props;
  const [count, setCount] = useState<number>(0);

  return (
    <>
      {users.map((user: User) => (
        <AnotherComponent
          key={user.id}
          username={user.name}
          onClick={() => onSelectUser(user.id)}
        />
      ))}
      {/* Some code */}
      <button onClick={onDelete} className={cn(s.button, { [s.disabled]: disabled }, className)}>
        {children}
      </button>
    </>
  );
};
```

## Project folder structure

Describes only core files and folders assuming that project uses routing.

`/src/` folder contains:
- `index.tsx`
- `index.css / index.scss`
- `_variables.scss` - use for global variables
- `types.ts` - use for global types and interfaces
- `const.ts` - use for global constants
- `functions.ts` - use for reusable functions. All the functions should be commented. In case of unexpected growth we prefer to create folder instead and separate functions to multiple files.
- `ui` - use for all react components. The folder contains such ones as: 
  - `pages` - use for components responsible for app pages.
  - `shared` - use for shared components.
  - `App.tsx`
- `modules` - use for modules implemented according the infrastructure, application and domain layers
  - `some-module-name`
    - `domain`
    - `application`
    - `infrastructure`
- `assets/svg` - use for `svg` files.

🚧 _Content is under development._
