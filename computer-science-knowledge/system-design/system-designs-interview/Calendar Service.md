![[Pasted image 20250421150202.png]]
### User Creates a New Event with Attendees
#### Event Service
- Receives the request and validates the input data (e.g., start time before end time, valid attendee emails/IDs).
- Starts a transaction with the **Primary Database**. d. Inserts the new `Event` record. e. Inserts `Attendee` records for the owner and each invited user (status `NEEDS_ACTION` for invitees, `ACCEPTED` for owner).
- Publishes messages to the **Message Queue** (e.g., Kafka): * `event_created` message (for Search Service, potentially others).
#### Notification Service
- Consumes `attendee_invited` messages from the Queue.
- For each message, looks up the recipient `User`'s notification preferences (from **Primary DB** or **Cache**).
- Formats the invitation notification 
- Sends the notification via the appropriate channel(s) (e.g., calls **Email Gateway**, **Push Notification Service**).
### User Views Their Calendar (Week View)
#### Calendar Query Service
- Checks the **Cache (Redis)** for cached results for this user/time range/timezone combination.
- If a valid cache entry exists, return it directly
- If not cached, queries the **Read Replica Database(s)** to fetch: * Non-recurring `Event` records where the user is an attendee, falling within the time range. * `RecurrenceRule` definitions for events where the user is an attendee. 
### User RSVPs to an Event Invitation
#### Event Service
- Validates the request (is the user actually an attendee of this event?).
- Starts a transaction with the **Primary Database**.
- Updates the `Attendee` record for this `user_id` and `event_id`, setting `rsvp_status` to "ACCEPTED" and updating `updated_at`. 
- Commits the transaction
- Publishes `attendee_status_updated` message to the **Message Queue** (containing `event_id`, `user_id`, new status).
### User Checks Free/Busy for Potential Meeting Attendees
#### Free/Busy Service
- Checks **Cache (Redis)** for cached free/busy results for this specific combination of users and time range. If valid cache entry exists, return it. _(Improves latency)_.
- Queries **Read Replica Database(s)** to fetch all relevant `Event` / `EventInstance` start/end times for _all_ requested users within the time range where their `rsvp_status` is `ACCEPTED` or `MAYBE` (potentially configurable).
- Merges the busy intervals for all users.
- Constructs the free/busy response format (e.g., a list of busy time blocks per user, or combined free slots).
- Stores the result in the **Cache** with a short TTL (free/busy data changes frequently).
- Returns the free/busy information back through the **API Gateway** to the **Client**.