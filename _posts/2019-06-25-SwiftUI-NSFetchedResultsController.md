---
layout: post
title: "Integrating NSFetchedResultsController with SwiftUI"
date: 2019-06-25
---

I know it's an unpopular opinion, but i've really grown to like Core Data. It has it's quirks, but it enables us to create a unidirectional data flow where all changes go through the managed object model, which drives the state of the view. If this already sounds like the perfect match for SwiftUIs reactive approach to rendering views, then that's mostly true. Core Data itself still isn't very swift-y, but we can leverage many new features in Swift 5.1 and Combine to create generic wrappers that can take any managed object and turn it into a view model that drives a SwiftUI view.

Let's start with `NSFetchedResultsController`. Subscribing to updates for many objects matching a fetch request has always been easier than subscribing to updates from a single managed object, thanks to `NSFetchedResultsController`. It comes with a delegate that informs us about changes to the underlying data in a structured way, because it was designed to integrate with tables and collection views. But we won't need most of that.

First we declare the view model.

```swift
class FetchedObjectsViewModel<ResultType: NSFetchRequestResult>:
    NSObject, NSFetchedResultsControllerDelegate, BindableObject {
```

There is already a lot to unpack here:

- The view model is a generic type with a type parameter `ResultType` that defines the kind of Core Data entity of its fetched results controller. Since `NSFetchedResultsController` fetches objects that conform to `NSFetchRequestResult` our `ResultType` is constraint accordingly.
- The view model is a subclass of `NSObject`, because it implements `NSFetchedResultsControllerDelegate`, which enables us to listen to updates from the fetched results controller, but also requires us to conform to `NSObjectProtocol`.
- Lastly the view model implements the `BindableObject` protocol from the SwiftUI framework, which enables us to communicate changes about its underlying data to the view.

This might sound a little complicated, but we're also almost done already! The rest is boilerplate.

The view model stores the fetched results controller it gets passed during Initialization, becomes its delegate to get informed about changes about the underlying data and starts querying Core Data:

```swift
    private let fetchedResultsController: NSFetchedResultsController<ResultType>
    
    init(fetchedResultsController: NSFetchedResultsController<ResultType>) {
        self.fetchedResultsController = fetchedResultsController
        // Should be called from subclasses of NSObject.
        super.init()
        // Configure the view model to receive updates from Core Data.
        fetchedResultsController.delegate = self
        try? fetchedResultsController.performFetch()
    }
```

It defines a `PassthroughSubject` "didChange" that doesn't pass any specific data and never fails:

```swift
    // MARK: BindableObject
    var didChange = PassthroughSubject<Void, Never>()
```

It implements the `controllerDidChangeContent` method of `NSFetchedResultsControllerDelegate` and calls the sends updates via "didChange" every time the underlying data changes:

```swift
    // MARK: NSFetchedResultsControllerDelegate
    func controllerDidChangeContent(_ controller: NSFetchedResultsController<NSFetchRequestResult>) {
        didChange.send()
    }
```

And since the backing fetched results controller is private to the view model, a little helper that ensures the view always receives an array, never `nil`:

```swift
    var fetchedObjects: [ResultType] {
        return fetchedResultsController.fetchedObjects ?? []
    }
}
```

Now we can create an `@ObjectBinding` to the view model and simply iterate over its `fetchedObjects` property to e.g. create a list item for each object. SwiftUIs `List` and `ForEach` require a unique identifier when passing an arbitrary array, and luckily instances of `NSManagedObject` already come with one called `objectID`.

```swift
struct ComicsView: View {

    @ObjectBinding var viewModel: FetchedObjectsViewModel<ComicEntity>

    var body: some View {
        List {
            ForEach(viewModel.fetchedObjects.identified(by: \.objectID)) { comic in
                ComicsItemView(viewModel: ManagedObjectViewModel(managedObject: comic))
            }
        }
    }

}
```

That last snippet already contains a teaser for the next post, which will be about creating view models for single instances of managed objects.
