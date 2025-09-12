# Problem Set 1
Meg Diulus (kerb: mediulus)

Jump to:
- [Excercise 1](#excercise-1)
- [Excercise 2](#excercise-2)
- [Excercise 3](#excercise-3)
- [Excercise 4](#excercise-4)

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

        concept PasswordAuthentication
            purpose limit access to known users
            principle 
                after a user registers with a username and a password,
                they can authenticate with that same username and password
                and be treated each time as the same user
            state
                a set of Users with …
            actions
                register (username: String, password: String): (user: User)
                …
                authenticate (username: String, password: String): (user: User)

1. Complete the definition of the concept state.

        state
        a set of Users with
            a username String
            a password String

2. Write a requires/effects specification for each of the two actions. (Hints: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)

        actions
            register (username: String, password: String): (user: User)
                requires: no existing user has the username = username
                effects: creates a User with the username=username and a password=password
            authenticate (username: String, password: String): (user: User)
                requires: a User exists with the username = username and their password=password
                effects: that user who matches the descriptions above

3. What essential invariant must hold on the state? How is it preserved?

    An essential invariant is that username is unique for all all Users. This is preserved in "register," if a person wants to register as another username that already, they would be denied. 

4. One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality. (Hints: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)

        concept PasswordAuthentication
            purpose limit access to known users
            principle 
                after a user registers with a username and a password, they are emailed a secret token
                they can authenticate with that same username and password
                and be treated each time as the same user
            state
                a set of Users with
                    a username String
                    a password String
                    a token String
                    a isConfirmed Flag
            actions
                register (username: String, password: String): (user: User, token: String)
                    requires: no existing user has the username = username
                    effects: creates a User with the username=username, a password=password, isConfirmed = false, and returns a token to be emailed
                authenticate (username: String, password: String): (user: User)
                    requires: a User exists with the username = username, their password=password, and isConfirmed=True
                    effects: return that User
                confirm(token: String, username: String) 
                    requires: a User exists with the username = username and that User has token = token
                    effects: sets isConfirmed = True

## Excercise 3

        concept PersonalAccessToken [User]
            purpose
                allow a user to authenticate to services and APIs without using their password, by creating, using, and revoking secret tokens
            principle
                a user may create one or more tokens;
                each token is a secret string generated by the system;
                presenting a valid token authenticates the user until it expires or is revoked;
                tokens can be individually deleted without affecting the user account or other tokens
            state
                a set of Tokens with
                    a token String
                    an owner User
                    an expiration Time
                    a scopes {String}
                    a note String
                    a revoked Flag

            actions
                create(owner: User, expiration: Time, scopes: {String}, note: String ): token: Token
                    requires: owner exists
                    effects: generates a secret token that it shows to the user once after creation
                             creates a new Toke:
                                token = secret
                                owner = User
                                expiration = expiration
                                scopes = scopes
                                note = note
                                revoked = False

                revoke(token: Token)
                    requires: token to exist and revoked = False
                    effects: sets revoked = True for that token

                authenticate(owner: User, scope: String, token: String): (user:User)
                    requires: there exists a token with token=token, expires > now, and revoked = False
                    effects: returns token.owner

In my opinion, GitHub should improve their page simply because these tokens are not truly like passwords. A password is a single, long-term credential that authenticates a user across their entire account, and if it is compromised the user must actively reset or change it. By contrast, a GitHub personal access token is designed to be more temporary and flexible: a user can create multiple tokens, each with its own scopes and expiration date, and tokens can be revoked individually without touching the account password. This makes them behave less like “another password” and more like disposable keys that can be granted or taken away as needed. Highlighting this distinction more clearly would help users understand that tokens provide better security practices—such as reducing the damage of a leaked credential—rather than simply being “a password by another name.”

## Excercise 4

### Confrence Room Booking

        concept ConferenceRoomBooking
            purpose Allow users to reserve conference rooms for specific times so that conflicts are avoided.
            principle
                A user may request a room for a continuous time interval.
                A booking is valid if and only if no existing booking overlaps the requested interval in the same room. 
                Only the user who created a booking may modify or delete it.

            state 
                a set of Rooms with
                    a roomNumber Number
                    a bookings {Booking}
                
                a set of Bookings with
                    a booker User
                    a startTime Time
                    a room Room
                    an endTime Time

                a set of Users with
                    a username String
                
                actions
                    bookRoom(booker: User, startTime: Time, endTime: Time, roomNumber: Room): (booking: Booking)
                        requires: 
                            Room number exists with that roomNumber
                            booker is an existing User
                            the endTime is after the startTime
                            no existing booking for the room overlaps with [startTime, endTime]
                        effects: creates and returns a new Booking with the parameters defined
                    
                    editBookingStart(booking: Booking, newStart: Time, user: User ): booking: Booking // wondering if the parameters for this should change to be like "Room, User, and newStart?"
                        requires: 
                            the booking to exist
                            user = booking.booker
                            newStart < booking.endTime
                            newStart does not overlap with other bookings for booking.room

                        effects: sets the booking startTime = newStart

                    editBookingEnd(booking: Booking, newEnd: Time, user: User ): booking: Booking // wondering if the parameters for this should change to be like "Room, User, and newEnd?"
                        requires: 
                            booking exists
                            user = booking.booker
                            newEnd > booking.startTime
                            newEnd does not overlap with other bookings for booking.room
                        effects: sets the booking endTime = newEnd

                    deleteBooking(booking: Booking, user: User)
                        requires:
                            booking exists
                            user = booking.booker
                        effects:
                            removes booking from booking.room


Notes: 
    Parameters for edits: It’s clearer and less error-prone to pass the Booking object itself (plus the acting User) rather than reconstructing it from (Room, User, Time). This way, you know exactly which booking is being edited.


### Billable Hours Tracking

        concept BillableHoursTracking 
            purpose Record work sessions per employee and project to enable accurate client billing.
            principle
                Each session belongs to exactly one employee and one project and spans a continuous time interval.
                An employee may have at most one active (open) session at a time.
                If an employee forgets to end a session, they must end it first, then may edit start/end to correct it.
                Sessions cannot have negative duration or overlap other sessions for the same employee.


            state
                a set of Employees with
                    a name String

                a set of Projects with
                    a code String
                    a displayName String

                a set of Sessions with
                    a employee Employee
                    a project Project
                    a note String
                    a startTime Time
                    a endTime Time?
                    a isActive Flag

            actions

                startSession(employee: Employee, project: Project, note: String, startTime: Time): (session: Session)
                    requires:
                        employee exists
                        project exists
                        no session exists with session.employee = employee and session.isActive = true
                    effects:
                        creates and returns a new Session with given fields, endTime = null, isActive = true


                endSession(employee: Employee, endTime: Time): (session: Session)
                    requires:
                        there exists an active session with session.employee = employee
                        endTime > session.startTime
                    effects:
                        sets session.endTime = endTime
                        sets session.isActive = false
                        returns session


                editSessionStart(session: Session, employee: Employee, newStart: Time): Session
                    requires:
                        session exists
                        employee = session.employee
                        session.isActive = false
                        newStart < session.endTime
                        changing start time does not cause overlap with any other session of employee
                    effects:
                        sets session.startTime = newStart
                        sets session.needsReview = true
                        returns session

                editSessionEnd(session: Session, employee: Employee, newEnd: Time): Session
                    requires:
                        session exists
                        employee = session.employee
                        session.isActive = false
                        newEnd > session.startTime
                        changing end time does not cause overlap with any other session of employee
                    effects:
                        sets session.endTime = newEnd
                        sets session.needsReview = true
                        returns session


Note: The user must end their session before editing the session, additionally there cannot be any overlapping sessions for an employee


### Time-Based One-Time Password

        concept Time-Based One-Time Password
            purpose
                Provide a second authentication factor in addition to a password,
                so that even if a password is compromised, the attacker still
                cannot log in without access to the user’s device.

            principle
                Each account shares a secret key with the server.
                Both the server and the user’s device use the current time window plus the secret to generate a numeric token.
                A token is valid only for that short window, limiting replay.

            state
                a set of Accounts with
                    a username: String
                    a secretKey: String

                system parameters with
                    a timeWindow: Number 

            actions
                registerUser(username: String): (account: Account)
                    requires: account does not already exist for the user
                    effects: creates an account for the user and assigns a secretKey

                generateToken(username: String, currentTime: Time): Number
                    requires: account exists for user
                    effects: uses account.secretKey and currentTime to compute a short-lived code

                checkCode(username: String, testingCode: Number, currentTime: Time): Boolean
                    requires: account exists for user
                    effects: verifies whether testingCode matches the valid token for that time window


[Back to Top](#problem-set-1)