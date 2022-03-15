## Architecture

The application is divided into several layers according to the clear architecture in order to ensure maintainability

### Domain layer

The layer contains entity definition and has no dependencies.

Each entity and relationships between entities are described by types and interfaces.
The layer has basic pure functions for working with entities and describing the general rules of the application.

### Application layer

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

### Infrastructure

This one contains all services implemented in accordance with the interfaces from the application layer.
We can use any libraries here like Axios, Mobx and etc.

The main advantage of the architecture is that if we need to replace Mobx with React we will change only this module. The rest of the application will not be affected.

Another advantage is that we can just copy the domain and the application layer to any application written in JS/TS, because the layers don't have any dependencies.

```txt
 ______         ____         _____________            ________________
| User |------>| UI |<----->| Application |<-------->| Infrastructure |
''''''''       ''''''       '''''''|'''''''          | Axios, MOBX ...|
                                   |                 ''''''''''''''''''
                             ______|______
                            |   Domain    |
                            '''''''''''''''
```

### Adapters

Adapters can be thought of as a layer that transforms data passed from one layer to another. It's mostly used to connect the application layer and the infrastructure.
For example when we have an object from the backend in snake case and have to transform it for the application layer, which uses camel case. 

## Modules

The application also can be divided into modules. Each module must implement the architecture described above. Any module has to be separated from other modules, any imports have to be avoid if it's possible.
