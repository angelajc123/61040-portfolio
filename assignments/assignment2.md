# Assignment 2
## Exercise 1: Reading a concept
1. One invariant is that the total number of purchases for an item is never greater than the number of requests. The other variant is that every purchase must be linked to a request in the registry. The latter is more important because this means we bought the giftee something they did not want, which defeats the point of the concept.
2. One way someone might purchase an item that does not have a request is if the giftee removes the item after the gifter has purchased it. To fix this, we can require that the item in removeItem has not been purchased by anyone before removing the item.
3. The concept does allow for registries to be open and closed repeatedly. This makes sense because there are various times of the year where you may be receiving gifts, like your birthday and then Christmas, and you may want to keep the same gift requests.
4. In practice this would not matter because you can just close the registry and it would be inactive. However, there could be a storage overload in time if you do not delete any of the registries from memory ever.
5. The registry owner is likely to query for items that have been purchased and who purchased themand gifters are likely to query for items that have not been purchased.
6. I would change the principle such that when the registry closes the recipient cannot see which items were purchased or who purchased them.
7. Generic types are good for design because they are meant to show the overall workflow, not the specific of how a concept will be implemented. For different systems, the item type may be different, but the workflow remains the same.

## Exercise 2: Extending a familiar concept
1. State - a set of Users with a username and a password
2. 
```````
register(username: String, password: String): (user: User)  
    requires username not already taken
    effects create a new user with this username and password

authenticate (username: String, password: String): (user: User)
    requires user exists with username and password
    effects returns user with username and password
```````

3. One invariant is that no two users can have the same username. This is preserved by requiring the username to not be taken when registering a new user.

4. 
```````
concept PasswordAuthentication  

purpose limit access to known users  

principle  
    after a user registers with a username and a password, they can authenticate with that same username and password and be treated each time as the same user  

state  
    a set of Users with 
        a username String
        a password String
        a confirmed Flag
    
    a set of Tokens with
        a User
        a token

actions  
    register (username: String, password: String): (user: User, token: Token)  
        requires username not already taken  
        effects create a new user with this username and password, create a new token for user
    
    confirm (token: Token)
        requires token exists
        effects confirm user with username and password, set confirmed flag to true

    authenticate (username: String, password: String): (user: User)  
        requires user exists with username and password  
        effects returns user with username and password 
```````
## Exercise 3: Comparing Concepts
```````
concept PersonalAccessToken

purpose 
    limit access to repositories

principle
    a user generates a personal access token. The system issues a random string and stores it. The user copies this token and uses it to authenticate. The system checks the token, expiration, and scope, and grants access accordingly.

state
    a set of Users with 
        a username String
        a set of Tokens
    
    a set of Tokens with
        a identifier String
        a name String
        an expiration Date
        a access Scope

actions
    generateToken(user: User): (token: Token)  
        requires user to exist  
        effects create a new token for user
    
    setToken(token: Token, name: String, expiration: Date, scope: String)
        requires token exists
        effects sets token values
    
    authenticate(user: User, token: Token): (access: Scope)
        requires user exists and token exists and token is not expired
        effects returns access to Scope
```````

Personal access tokens are different because they require the user to already have an account, and the purposes is to limit or grant access to repositories. Personal access tokens can have different scopes and can expire. GitHub may write in their documentation use cases for personal access tokens for clearer usage.

## Exercise 4: Defining familiar concepts
### URL Shortener
```````
concept URLShortener

purpose
    limit the length of a URL

principle
    a user registers an either user-defined or auto-generated short URL that corresponds with a long URL, afterwards the user can get the long URL from the short URL.

state
    a set of URLs
        a long URL String
        a short URL String

actions
    registerShortURL(longURL: String, shortURL: String): (url: URL)
        requires shortURL to not exist
        effects takes user-defined short URL or generates a unique short URL, creates a new URL with longURL and short URL
    
    getLongURL(shortURL: String): (longURL: String)
        requires shortURL to exist
        effects returns corresponding longURL
```````
### Billable Hours Tracking
```````
concept BillableHoursTracking

purpose
    automate record keeping for billable hours

principle
    a company registers a client and their projects, employees log sessions on projects, and the system tracks the number of hours worked on for a client.

state
    a set of Clients with
        a set of Projects

    a set of Projects with
        a set of Sessions
    
    a set of Sessions with
        a startTime Number
        an endTime Number
        a done Flag

actions
    registerClient(client: Client)
        requires client to not exist
        effects creates a new client

    registerProject(project: Project)
        requires project to not exist
        effects creates a new project

    beginSession(project: Project, startTime: Number)
        requires project exists
        effects creates a new session starting at startTime, done is false

    endSession(session: Session, endTime: Number)
        requires session exists and is not done
        effects ends session at endTime, sets done to true
    
    autoEndSession(session: Session, timeout: Number)
        requires session exists and is not done
        effects ends session at timeout, sets done to true
    
    getSessions(client: Client):
        requires client exists
        effects returns all sessions associated with a client
```````
### Conference Room Booking
```````
concept ConferenceRoomBooking

purpose
    prevent double booking rooms

principle
    a company/university makes slots available for different rooms at various times. Employees/students can reserve a slot and are assured that they can meet in that room at that time.

state
    a set of Slots with
        a Room
        a Time
    
    a set of Bookings with
        a User
        a Slot

actions
    createSlot(r: Room, t: Time)
        effect creates a new slot associated with time t in room

    deleteSlot(s: Slot)
        requires no bookings for this slot
        effects remove s from set of slots

    reserve(u: User, t: Time, r: Room): Booking
        requires an unreserved slot for r at time t
        effects add a new booking for slot with user

    cancel(b: Booking)
        requires booking exists
        effects remove b from bookings
```````
