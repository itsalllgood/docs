# Version 1.10.0 (2017-05-04)
New features:

- GitHub integration now generally available for enterprise users. Self-setup interface completed.

# Version 1.9.0 (2017-03-07)
New features:

- View matching messages in the job writing interface when a message filter trigger is selected.
- "Tree view" exposed for job expression viewing. With valid syntax, you're able to see your expression as a syntax tree as we step slowly towards a more point-and-click interface.

# Version 1.75.0 (2016-12-08)

New features:

- Hold control while clicking on navigation buttons to open the target in a new window.
- Filter messages in your inbox by their content by selecting a message-filter trigger.

# Version 1.7.0 (2016-12-05)
#### _1.7 is all about user experience!_
New features:

- Material design—more whitespace and cleaner lines.
- Goto page on inbox and activity tables—save time when processing errors.
- Change number of items per page on inbox and activity table—care with this one on slow connections!
- Go to next or previous message or run—makes working through an audit trail easier
- Change user profile settings without changing password
- Select syntax style for code editors in user settings—clouds midnight is my new favorite
- Filter projects, jobs, triggers by name—on the fly for quick navigation
- Add adaptor logos to credentials list—quick identification
- Specify connection types on "Apps" list—seems there was some confusion about this. I know we're missing plenty of apps that have good APIs. Will consider logging API documentation as well.
- Shift second top-nav to a collapseable "side nav"—better use of screen real-esate.
- Use 'masonry' packing module for jobs, triggers, credentials, and project settings boxes—more efficient use of space
- Add material design to *this* documentation page!


# Version 1.6.0 (2016-11-24)

New features:

- Updated payment receipts to include project names.
- Added `update(...)` to Salesforce adaptor v0.3.0
- Added `fetchWithErrors` to HTTP adaptor v0.3.1

**New Salesforce helper function `update(...)`:** It takes an object and, so long as you're using the "Id" only updates.

```js
update("Patient__c", fields(
  field("Id", dataValue("pathToSalesforceId"),
  field("Name__c", dataValue("patient.first_name")),
  field(...)
))
```

**New http helper function `fetchWithErrors(...)`:** This function will perform a get request on an endpoint and return the response to another endpoint, regardless of whether the first GET suceeded or failed. It's currently being used to send message receipt confirmations back to a system of origin that uses OpenFn as an intermediary between it and an SMS gateway. If the SMS message doesn't get delivered because the phone number is invalid, we'd like that information the return all the way to Salesforce, rather than erroring out and staying in OpenFn.

```js
// =============
// We use "fetchWithErrors(...)" so that when the
// SMS gateway returns an error the run does not "fail".
// It "succeeds" and then delivers that error message
// back to Salesforce with the "Update SMS Status" job.
// =============
fetchWithErrors({
  "getEndpoint": "send_to_contact",
  "query": function(state) {
      return {
        "msisdn": state.data.Envelope.Body.notifications.Notification.sObject.SMS__Phone_Number__c,
        "message": state.data.Envelope.Body.notifications.Notification.sObject.SMS__Message__c,
        "api_key": "some-secret-key"
      }
  },
  "externalId": state.data.Envelope.Body.notifications.Notification.sObject.Id,
  "postUrl": "https://www.openfn.org/inbox/another-secret-key",
})
```

# Version 1.5.0 (2016-10-05)

New features:

- Delete credentials
- Delete triggers
- Archive jobs
- Continual testing from status.openfn.org

**Delete credentials and triggers:** Users can now delete credentials and triggers.

**Archive jobs:** Users can now archive jobs, rendering them inactive. Click "view archived jobs" to see and restore old jobs.

**status.openfn.org**: is now live, providing continual testing of key OpenFn services. We run both message-filter-based and timer-trigger-based jobs every five minutes to ensure availability, as well as measuring the round-trip time (in ms) that it takes for a server in a different geographical location to send valid JSON to OpenFn then receive and process the 200 response. (This time will vary according to the location of your servers, but it's important to note that we test the full round trip. Our servers typically send out 200s in about 5-6ms, but you can expect the round trip to complete in closer to 750ms.)

# Version 1.4.0 (2016-09-26)

New features:

- Run "matches" directly from your inbox view.
- Always display the latest notification, dismiss to scroll back in time.
- Login and signup server responses

**Run "matches" from inbox:** Users can now run matches in a single click from their inbox, getting notifications that runs have successfully started without having to navigate to the Message Inspector page for a given message. Look for the blue "play" button next to each match. Simply click to start running that job with the message in question.

**Latest notifications:** User notifications will now be displayed *newest-on-top* and when there are multiple stacked notifications users will be... well... notified. Click the small "x" to dismiss the latest notification, moving backwards in time until all have been read.

**Login/signup errors:** Until now, invalid login messages and duplicate singup emails had been only displayed in your brower's logs. (That's our fault.) You'll now see a handy "invalid credentials" or "email already registered" message when trying to log in or sign up.

# Version 1.3.0 (2016-09-20)

- New version of language-salesforce allows users to `alterState` with a custom function.

**alterState:** [documentation](https://openfn.github.io/docs/documentation/#alterstate-alter-state-to-make-sure-data-is-in-an-array)

# Version 1.2.0 (2016-09-15)

- Users can now select specific adaptor versions for their jobs.
- Jobs will "auto-upgrade" unless locked to a specific version.

**Adaptor versions:**  This means that the code beneath your job, once saved with a specific adaptor version, will never change. This is an important step forward for the whole community, as it enables more rapid progress—especially considering the growing number of outside contributors—without risking introducing instability to existing jobs.

Each new version of an adaptor will have release notes introducing the new features or changes to helper functions. To allow easy upgrades, we will still mandate that all new versions are backwards compatible.

# Version 1.1.0 (2016-08-29)

New features:

- Users can now run jobs based on **timers** as well as filters.
- Users can now view logs for all runs, not just the most recent.
- Jobs are "aware" of their last running state.
- `get(...)` and `post(...)` are now supported using the language-http adaptor, allowing users to make their own HTTP calls in jobs.


**Timer triggers:** On the triggers tab, users can set the trigger type to "timer" and input a whole number of seconds for the "interval". Any "active" jobs associated with this trigger will run periodically after the interval elapses.

**View logs for all runs:** By clicking on an individual run from either the Activity tab or the Message Inspector, users can view the full logs for that run, regardless of whether or not a more recent run took place with the same job and message.

**Job state:** When a job runs based on a timer, not an incoming message, it will preserve it's state for the next run. This feature is commonly used by language packs like language-surveycto, language-odk, and others to create a "cursor" to offset or limit database queries.

> For example, `fetchSubmissions(...)` in the language-surveycto adaptor takes three arguments: `formId`, `afterDate`, and `postUrl`. The first time this job runs it will only fetch submissions *after* the `afterDate`. If any submissions are received, it will take the last submission from the array (by date) and persist it in the `job_state` as `lastSubmissionDate`. The next time this job runs, say, 300 seconds (5 minutes) later, it will ignore `afterDate` and instead fetch submissions after `lastSubmissionDate`. While this particular helper function is very abstract (it does this one thing well) it's possible to write a job that simply alters the final "state" before completing, passing whatever data you'd like from *THIS RUN* to the *NEXT RUN* of the job.

**get(...) and post(...):** Have a look at this complex job using language-http. See how it is possible to provide a query and a callback for `get` while `post` takes a url and a body object. At the end, the user is setting state.lastSubmissionDate to `submissions[submissions.length-1].SubmissionDate`.

See the functions themselves at [language-http](https://github.com/OpenFn/language-http/blob/master/src/Adaptor.js).

```js
get("forms/data/wide/json/someForm", {
  query: function(state) {
    return { date: state.lastSubmissionDate || "Aug 29, 2016 4:44:26 PM"}
  },
  callback: function(state) {
    // Pick submissions out in order to avoid `post` overwriting `response`.
    var submissions = state.response.body;
    // return submissions
    return submissions.reduce(function(acc, item) {
        // tag submissions as part of the "someForm" form
        item.formId = "someForm"
        return acc.then(
          post(
            "https://www.openfn.org/inbox/some-inbox-uuid",
            { body: item }
          )
        )
      }, Promise.resolve(state))
      .then(function(state) {
        if (submissions.length) {
          state.lastSubmissionDate = submissions[submissions.length-1].SubmissionDate
        }
        return state;
      })
      .then(function(state) {
        delete state.response
        return state;
      })
  }
})
```
