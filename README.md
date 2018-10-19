# VanArsdel Inventory Sample

[![Build status](https://ci.appveyor.com/api/projects/status/s1gccf5kya2u0m7f?svg=true)](https://ci.appveyor.com/project/rido-min/inventorysample)

This repo showcases a sample Windows 10 application (using the Universal Windows Platform) focusing on Line of Business scenarios, showing how to use the latest Windows capabilities in desktop applications. The sample is based around creating and managing customers, orders and products for the fictitious company VanArsdel.

![VanArsdel Inventory Preview](docs/Introduction/img/app-screenshot.png)

## Features
This sample highlights:
- using [.NET Standard](https://docs.microsoft.com/en-us/dotnet/standard/net-standard)
- the master/details UI pattern
- forms layouts
- using the repository pattern to connect to SQLite
- example of Multiples views of the app

## Supported SDKs
- Fall Creators Update (16299)

## Documentation
Both the code and the documentation for this sample are under development. You can always find the latest docs [here](docs/README.md).

## Feedback and Requests
Please use GitHub Issues for bug reports and feature requests. For feature requests, use the Enhancement label. For bugs, use the Bug label.

## Contributing
Do you want to contribute? We would love to have you help out. Here are our [contribution guidelines](Contributing.md).

## License
This code is distributed under the terms and conditions of the MIT license.

## Supported By
This project is supported by the [.NET Foundation](http://dotnetfoundation.org/).

# Architecture Overview
**VanArsdel Inventory Sample** is based on an MVVM architecture pattern to facilitate the separation of the user interface from the application's business logic. You can read more about the MVVM pattern in the MVVM section of this documentation.

The following diagram shows the different layers used in the application.

![Architecture Diagram](docs/Architecture/img/ovw-layers.png)

# Views
Views are essentially what the user sees on the screen that allows them to interact with the application. Some examples of views in this application are: CustomersView, OrdersView and ProductsView.

Views contain sections and controls to display the user interface. The sections organize the controls in the view, and the controls show the information to the user. All views depend on a view-model that manages the logic of the user interface.

When you examine a view's code, you will notice that the first thing it does in the constructor is instantiate the view-model. Another thing to notice is the overridden methods: OnNavigatedTo and OnNavigatingFrom.

The OnNavigatedTo method is executed when the user navigates to the view and is used to initialize the view-model with the parameters received from the navigation process.

The OnNavigatingFrom method is executed when the user navigates away from the view and is used to free any resources used by the view-model.

The following code shows a typical implementation of a view.

```csharp
    public class CustomersView : Page
    {
        public CustomersView()
        {
            ViewModel = ServiceLocator.Current.GetService<CustomersViewModel>();
            InitializeComponent();
        }

        public CustomersViewModel ViewModel { get; }

        protected override async void OnNavigatedTo(NavigationEventArgs e)
        {
            ViewModel.Subscribe();
            await ViewModel.LoadAsync(e.Parameter as CustomerListArgs);
        }

        protected override void OnNavigatingFrom(NavigatingCancelEventArgs e)
        {
            ViewModel.Unload();
            ViewModel.Unsubscribe();
        }
    }
```

In order to simplify development and make the code more readable, views are subdivided into subviews.

For example, the CustomersView consists of the following subviews:
-	**CustomersList** – contains:
    -  the list of customers
    -  controls to execute common actions over the collection of customers, such as:
       -  Add New Customer
       -  Search Customers
       -  Refresh Customer List
-	**CustomersDetails** – contains:
    -    the details of the selected customer
    -    input controls to enable the editing of the customer's details
    -    common actions available for a customer, such as: Edit and Delete
-	**CustomersCard** – shows the main properties of the selected customer as a quick and read only view
-	**CustomersOrders** – contains the list of orders associated with the selected customer
-	**CustomersView** – the top-level view containing all the subviews described above

The following image shows the different subviews contained in the CustomersView.

![Views and Subviews](docs/Architecture/img/ovw-views-subviews.png)

## ShellView
A ShellView is a special type of view. It serves as a container for other views. It contains a frame that enables navigation between other views, and a status bar on the bottom to display messages to the user.

Any time a new window is opened in the application, a new ShellView is created and the content is initialized using the frame.

## MainShellView
The application's main window uses a specialized version of a ShellView; the MainShellView.

The MainShellView is the same as any other ShellView, but it contains a navigation pane on the left to provide the user with ways to navigate to different views.

When the user executes the application and signs in, the MainShellView is created. There can be multiple ShellViews in the application, but there can be only one MainShellView.

The following image identifies the different elements in the MainShellView.

![MainShellView](docs/Architecture/img/ovw-views-mainshell.png)

# ViewModels
View-models are another essential part of the MVVM architecture pattern. You can read more details about the part a view-model plays in the ViewModel Hierarchy section of this documentation.

The view-model contains the UI logic of the application, so it is reasonable to believe that there will be at least one view-model for each View. In some cases, where the view requires more complexity, there may be more than one view-model for that view.

The Customers view is associated with a Customers view-model, and this view-model references both the CustomersList view-model and the CustomersDetails view-model.

The following diagram shows this relationship.

![ViewModels Relationships](docs/Architecture/img/ovw-viewmodels-relationships.png)

## ViewModel Hierarchy
Most of the properties and methods are common for all the different view-models, so inheritance is used to reuse this functionality.

The following diagram shows the view-models hierarchy.

![ViewModels Hierarchy](docs/Architecture/img/ovw-viewmodels-hierarchy.png)

### ObservableObject
The ObservableObject is the base class for all the objects that need to notify that one of its properties has changed. It contains a typical implementation of the INotifyPropertyChanged interface.

All view-model and model classes inherit from this object.

### ViewModelBase
This is the base class for all view-models in the application. It contains common members used by all view-models. It also contains a reference to the common services used by most of the view-models. These services are:
-	NavigationService
-	MessageService
-	DialogService
-	LogService
-	ContextService

### GenericListViewModel
This is a generic class used by all the view-models that need to manage a list of elements. It contains common functionality used to query, select, add or delete elements from a data source. This functionality includes the following members:
-	Items – generic list of items
-	ItemsCount – number of items in the list
-	SelectedItem – item that is currently selected
-	SelectedItems – list of items that are currently selected (when in multi-selection mode)
-	SelectedIndexRanges – list of ranges of items that are currently selected (when using a virtual collection)
-	Query – text used to query and retrieve items from the data source
-	OnRefresh – abstract method executed to retrieve the list of items
-	OnNew – abstract method executed when the “New Item” command is invoked
-	OnDeleteSelection – abstract method to delete items that are currently selected

Refer to the source code of GenericListViewModel for a detailed list of members implemented in this class.

### GenericDetailsViewModel
This is a generic class used by all the view-models. It contains the details of a single item, as well as common functionality used to edit and delete an item. This functionality includes the following members:
-	Item – a generic item to be shown or modified
-	OnEdit – executed when editing the item starts
-	OnCancel – executed when editing the item is canceled
-	OnSave – executed when the item's changes are about to be saved
-	OnDelete – executed when the item is going to be deleted

Refer to the code of GenericDetailsViewModel for a detailed list of members implemented in this class.

# Models
Models are the third leg in the MVVM architecture pattern. You can read more details about the concepts of the Model in the MVVM – Models section of this documentation.

In this application, a model wraps the data of a business object to better expose its properties to the view. In other words, the business object is the raw data received from the data source while the Model is a “view friendly” version, adapting or extending its properties for a better representation of the data.

For example, the Customer business object contains the properties “Name” and “Last Name” and the CustomerModel exposes a “FullName” property which is a concatenation of these two properties, for display in the customer details view.

In a more complex scenario, the Customer business object contains only the “CountryCode”, while the CustomerModel also exposes a “CountryName” property which is updated from a lookup table if the user selects a new country in the ComboBox control.

The Model also helps decouple the business objects used in the data layer from the classes used in the presentation layer. So if a change is required in a business object's schema, the application is less likely to be affected.

# Services
View-models make use of Services to execute the operations requested by the user, such as create, update or retrieve a list of customers or products. View-models can also make use of Services to log user activity, show dialogs or display a text in the status-bar.

Services contain the core functionality of the application. This functionality is split into two types of services:

-	**Application Services** – implement core functionality needed by the infrastructure of the application (independent of the business of the application and can be reused in other applications)
-	**Domain Services** – implement the business-specific functionality of the application (also known as Business Services)

The following diagram shows the two groups of services used in this application:

![Service Groups](docs/Architecture/img/ovw-services-groups.png)

## Application Services
Here is a brief description of the Application Services used in this application:

| Service                | Description |
|------------------------|-------------|
| **Navigation Service** | Exposes the functionality to navigate to a different view (either forward to a specific view, or back to the previous view). It also offers the possibility to open a view in a new window. |
| **Message Service** | Enables communication between different components of the application, without them having to know anything about each other. The communication between components is based on a publisher-subscribers pattern. |
| **Log Service** | Exposes the methods to write log entries to a local repository (to log user activity for debugging or auditing purposes).
| **Login Service** | Implements the authentication and authorization mechanisms (allows users to access the application). |
| **Dialog Service** | Exposes methods to display a dialog message (to show information, or ask for confirmation). |
| **Context Service** | Exposes properties and methods related to the current execution context. This service is used to manage a multi-window environment where each window is executed in a different thread. |

## Domain Services
The domain services in this application provide CRUD operations (Create, Read, Update, Delete) over the business entities. There is a specific service for Customers, Orders, OrderItems and Products.

For example, the Customer service supplies the following methods:
-	**GetCustomer(id)** – gets a single customer (using the 'id' parameter)
-	**GetCustomers(request)** – gets a collection of customers (using the 'request' parameter)
-	**GetCustomers(skip, take, request)** – same as GetCustomers(request), but excludes the first ‘skip’ customers, and returns the next ‘take’ customers
-	**GetCustomersCount(request)** – returns the number of Customers (using the 'request' parameter)
-	**UpdateCustomer(customer)** – updates, or creates a new Customer (using the values contained in the 'customer' parameter)
-	**DeleteCustomer(customer)** – deletes a Customer (specified by the 'customer' parameter)

There is also a LookupTables Service used to retrieve information for common tables such as Categories or CountryCodes. For example, this service is used to get the name of a country from its code, or the tax rate for a specific tax type.

# Services and Dependency Injection
When we need to make use of a service, the first thing we need is a reference to that service. The easiest way to get a reference to a service would be just creating an instance of the required service where it's needed.

For example, let’s say we need to write log entries using the Log Service:

```csharp
    var logService = new LogService();
    logService.Write(message);
```

Simple and straightforward, but this approach could lead to further problems:
1.	We are assuming that the Log Service is implemented in the scope of the current library, but this is not always the case. In this application, most of the services used by the view-models are implemented in a different library which is not referenced by the view-models library.
2.	We are creating a new instance to write to a log, but wouldn’t it be better to reuse the same instance everywhere in the application? The answer depends on several factors and may be subject to changes in the future. In any case, the decision on how to create an instance of a service should be made outside of the component that uses the service.
3.	On the other hand, creating a hard-coded reference to a service is also a bad idea if we are planning to test our application. When testing a component, we will need a mechanism to replace one or more services with a fake implementation, in order to trace and check if the component iteself is working as expected.

These and other issues can be solved by using the Dependency Injection pattern. With Dependency Injection, we get an instance of a service by using a ServiceLocator.

> **Note:**
> The ServiceLocator implemented in this application is based on the Microsoft.DependencyInjection.Abstractions package library.

To get an instance of a service using a ServiceLocator it can be as simple as the following code:

```csharp
    var logService = ServiceLocator.Request<ILogService>()
```

The first thing to note is that we are not creating an instance of a service, **we are requesting an instance of a service by specifying an interface**.

This has the following considerations:
-	the ServiceLocator may return a new instance, or reuse an existing instance of the service (singleton)
-	the ServiceLocator may return a real implementation of ILogService, or a fake implementation for testing purposes
-	since we are requesting the service via an interface, the service can be implemented in another library, out of the scope of the current component

Whether the ServiceLocator returns a new instance, an existing instance, a real implementation or a fake implementation depends on the configuration of the service in the ServiceLocator. We will talk about the ServiceLocator configuration later.

The following diagram shows the relationship between components using a ServiceLocator.

![Dependency Injection ServiceLocator](docs/Architecture/img/ovw-servicelocator-refs.png)

As you can see in the diagram, the component consuming the ILogService can make use of the LogService, even when the service is implemented out of the scope of the component.

Dependency Injection is not only used to resolve service instances, it is also used to request view-models, and can be used for any object in the application. We just need to tell the ServiceLocator how to resolve the request.

## ServiceLocator Configuration
We saw before how to request a service using the ServiceLocator. Now let’s see how to configure the ServiceLocator to return a specific instance when we request a service.

The ServiceLocator uses a ServiceCollection class where services are registered and configured to be used in the application.

>**Note:** The ServiceCollection class is provided by the Microsoft DependencyInjection Library.

To register a service in the ServiceCollection we can use the following code:
```csharp
    serviceCollection.AddSingleton<ILogService, LogService>();
```

In this case, we are registering the ILogService interface to use the LogService implementation as a singleton. This means that the first time we request an ILogService, the ServiceLocator will return an instance of the LogService class, and will then reuse the same instance for all the future requests.

If we want the ServiceLocator to return a new instance for every request, we can register the ILogService as transient:
```csharp
    serviceCollection.AddTransient<ILogService, LogService>();
```

Registering a service as transient means that we always want a new instance of the service implementation.

In this application, all services are configured as singletons, but there is one exception to this rule: The Navigation service.

The Navigation service is used to navigate to another view in the application. In a single window environment, the Navigation service can be reused by all components of the application, but in a multi-window environment, each window needs its own instance of the service.

To solve this scenario, the Microsoft DependencyInjection Library introduces the concept of scopes and scoped services. We can see a scope as a context of execution. The following code configures a service as scoped.
```csharp
    serviceCollection.AddScoped<INavigationService, NavigationService>();
```

Scoped services are similar to singleton services but only when requested in the same scope. Requesting a scoped service in the same scope will return always the same instance, but requesting the service in a different scope will return a different instance.

As developers, we are responsible for creating a new scope when necessary. In this application, a new scope is created for each new window opened in the application. This way, when we request an INavigationService in one window we will receive a Navigation service to navigate in that window.

## Declaring Dependencies in the Constructor
When using the Dependency Injection pattern it is very common to declare the required services in the constructor of the class.

For example, the Customer service depends on the ILogService and its constructor is declared as follows:

```csharp
    public CustomerService(ILogService logService)
    {
        LogService = logService;
    }

    private ILogService LogService { get; }
```

When we request a CustomerService, the ServiceLocator examines the constructor and tries to resolve its dependencies. Since we configured the ServiceLocator to resolve ILogService to LogService, it will obtain a reference to an instance of LogService and pass it to the constructor.

Dependency Injection is not only used to resolve services, it is also used to resolve view-models and in general it could be used to resolve the instantiation of any class used in the application when required.

> **Note:**
> Only classes registered in the ServiceCollection can be resolved by using the ServiceLocator.

# Summary
The VanArsdel Inventory Sample architecture is designed following the MVVM pattern, separating the user interface from the application's business logic:
-	Views represent the user interface of the application
-	ViewModels contains the user interface logic
-	Models represent the business data of the application

View-models make use of services to execute the operation requested by the user. We make a distinction between two kinds of services:
-	Domain services – implement the business functionality of the application
-	Application services – implement core functionality needed by the infrastructure of the application

Following the Dependency Injection pattern, both view-models and services are instantiated using a ServiceLocator.
-	we request a service from the ServiceLocator by specifying an interface
-	the ServiceLocator can be configured to instantiate services as Singleton, Transient or Scoped
