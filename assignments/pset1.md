# Problem Set 1
Meg Diulus (kerb: mediulus)

## Excercise 1

    concept GiftRegistration [User, Item]

    purpose: track purchases of requested gifts
    principle
        a recipient creates a registry, and adds items to it indicating the number of each requested;
        opens the registry so it becomes publicly visible;
        then givers can view which items are still available and purchase them;
        and finally the recipient closes the registry, after which it is no longer publicly visible
        but the recipient can see which items were purchased and by whom.

    state
        a set of Registrys with
            an owner User
            an active Flag
            a set of Requests

        a set of Requests with
            an Item
            a count Number
            a set of Purchases

        a set of Purchases with
            a purchaser User
            an Item
            a count Number

    actions
        create (owner: User): (registry: Registry)
            effects create a new registry with this owner, active set to false and no requests

        addItem (registry: Registry, item: Item, count: Number)
            requires registry exists
            effects if a request for this item exists, add the count
            otherwise create a new request for the item with this count and add to registry

        removeItem (registry: Registry, item: Item)
            requires a request for this item exists in the registry
            effects remove the request from the registry

        open (registry: Registry)
            requires registry exists and it is not active
            effects make registry active

        close (registry: Registry)
            requires registry exists and it is active
            effects make registry not active

        purchase (purchaser: User, registry: Registry, item: Item, count: Number)
            requires registry exists, is active and has a request for this item with at least count
            effects create a new purchase for this purchaser, item and count
            and decrement the count in the matching request


1. **Invariants.** What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

    The first invariant is that the count of a request should always be greater than or equal to zero (request.count ≥ 0). The second invariant is that every Purchase must be attached to a Request (ie. there can not be any orphan purchases)

    The second invariant is more important because the purchases loose context, counts may also be effected, but it corrupts the purchase -> Request -> Registry subsequence. Because of this, purchase is most affectted by it and preserves the link by creating it and linking the new Purchase to the Request. 

2. **Fixing an action.** Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

    removeItem() potentially breaks this invariant because it could delete a Request that has purchases, this would orphan the purchases of this item. This problem can be fixed by adding that pre-condition of "Request.purchases == {}."

3. **Inferring behavior.** The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

    A registry can be opened and closed repeatedly because of how "open" and "close" work. They do not create or destroy the Registry, the only set whether or not it is active to True or False. This is a good design decision because a user may want to have the same registry for another event. For instance a user may get married and have a baby shortly after and want a similar Registry, so rather than having to create an entire new one, they could simply re-open the Registry. 

4. **Registry deletion.** There is no action to delete a registry. Would this matter in practice?
    I would say it does matter in practice. For two primary reasons, first that if the application is giant, your data base could get extremely cluttered over time. You do not need a registry from say--20 years ago (even 5 years ago). Second, users may worry about privacy issues and not want that information to be stored and never deleted. 

5. **Queries.** What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)

    A query for the giver of the gift would be isItemAvailable for purchase. This would essentially tell whether or not the count of the item was greater than or equal to zero. A query for the registry owner would be checkStatus of the registry to potentially see how many items have been purchased, need to be purchased, etc. 

6. **Hiding purchases.** A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?
    
    To augment the concept to support this update, I would add a "hidden" attribute in the Registry concept that defines whether or not the owner can view it. Then either in "create" or "open" I would add a "hidden" parameter. Probably in "open" that way there remains flexibilty in the concept.

7. **Generic types.** The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

    Using a generic Item type (like an SKU) means each product has a stable code that doesn’t change if its name, description, or price does. This keeps the registry simple, while full product details can be looked up separately when needed.

## Excercise 2

1. Complete the definition of the concept state.

    Answer

2. Write a requires/effects specification for each of the two actions. (Hints: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)

    Answer

3. What essential invariant must hold on the state? How is it preserved?

    Answer

4. One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality. (Hints: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)

    Answer

## Excercise 3

### Passwords vs. Personal Access Tokens

## Excercise 4

yabetse