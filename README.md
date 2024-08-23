# Loops Ruby SDK

## Introduction

This is the official Ruby SDK for [Loops](https://loops.so), an email platform for modern software companies.

## Installation

Install the gem and add it to the application's Gemfile like this:

```bash
bundle add loops_sdk
```

If bundler is not being used to manage dependencies, you can install the gem like this:

```bash
gem install loops_sdk
```

## Usage

You will need a Loops API key to use the package.

In your Loops account, go to the [API Settings page](https://app.loops.so/settings?page=api) and click **Generate key**.

Copy this key and save it in your application code (for example, in an environment variable).

See the API documentation to learn more about [rate limiting](https://loops.so/docs/api-reference#rate-limiting) and [error handling](https://loops.so/docs/api-reference#debugging).

In an initializer, import and configure the SDK:

```ruby config/initializers/loops.rb
require "loops_sdk"

LoopsSdk.configure do |config|
  config.api_key = 'your_api_key'
end
```

```ruby
begin

  response = LoopsSdk::Transactional.send(
    transactional_id: "closfz8ui02yq......",
    email: "dan@loops.so",
    data_variables: {
      loginUrl: "https://app.domain.com/login?code=1234567890"
    }
  )

  render json: response

rescue LoopsSdk::APIError => e
  # Do something if there is an error from the API
end
```

## Default contact properties

Each contact in Loops has a set of default properties. These will always be returned in API results.

- `id`
- `email`
- `firstName`
- `lastName`
- `source`
- `subscribed`
- `userGroup`
- `userId`

## Custom contact properties

You can use custom contact properties in API calls. Please make sure to [add custom properties](https://loops.so/docs/contacts/properties#custom-contact-properties) in your Loops account before using them with the SDK.

## Methods

- [ApiKey.test()](#apikey-test)
- [Contacts.create()](#contacts-create)
- [Contacts.update()](#contacts-update)
- [Contacts.find()](#contacts-find)
- [Contacts.delete()](#contacts-delete)
- [MailingLists.list()](#mailinglists-list)
- [Events.send()](#events-send)
- [Transactional.send()](#transactional-send)
- [CustomFields.list()](#customfields-list)

---

### ApiKey.test()

Test if your API key is valid.

[API Reference](https://loops.so/docs/api-reference/api-key)

#### Parameters

None

#### Example

```ruby
response LoopsSdk::ApiKey.test
```

#### Response

This method will return a success or error message:

```json
{
  "success": true,
  "teamName": "Company name"
}
```

```json
{
  "error": "Invalid API key"
}
```

---

## Contacts.create()

Create a new contact.

[API Reference](https://loops.so/docs/api-reference/create-contact)

#### Parameters

| Name            | Type   | Required | Notes                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `email`         | string | Yes      | If a contact already exists with this email address, an error response will be returned.                                                                                                                                                                                                                                                                                                                            |
| `properties`    | object | No       | An object containing default and any custom properties for your contact.<br />Please [add custom properties](https://loops.so/docs/contacts/properties#custom-contact-properties) in your Loops account before using them with the SDK.<br />Values can be of type `string`, `number`, `nil` (to reset a value), `boolean` or `date` ([see allowed date formats](https://loops.so/docs/contacts/properties#dates)). |
| `mailing_lists` | object | No       | An object of mailing list IDs and boolean subscription statuses.                                                                                                                                                                                                                                                                                                                                                    |

#### Examples

```ruby
response = LoopsSdk::Contacts.create(email: "hello@gmail.com")

contact_properties = {
  firstName: "Bob" /* Default property */,
  favoriteColor: "Red" /* Custom property */,
};
mailing_lists = {
  cm06f5v0e45nf0ml5754o9cix: true,
  cm16k73gq014h0mmj5b6jdi9r: false,
};
response = LoopsSdk::Contacts.create(
  email: "hello@gmail.com",
  properties: contact_properties,
  mailing_lists: mailing_lists
)
```

#### Response

This method will return a success or error message:

```json
{
  "success": true,
  "id": "id_of_contact"
}
```

```json
{
  "success": false,
  "message": "An error message here."
}
```

---

## Contacts.update()

Update a contact.

Note: To update a contact's email address, the contact requires a `userId` value. Then you can make a request with their `userId` and an updated email address.

[API Reference](https://loops.so/docs/api-reference/update-contact)

#### Parameters

| Name            | Type   | Required | Notes                                                                                                                                                                                                                                                                                                                                                                                                               |
| --------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `email`         | string | Yes      | The email address of the contact to update. If there is no contact with this email address, a new contact will be created using the email and properties in this request.                                                                                                                                                                                                                                           |
| `properties`    | object | No       | An object containing default and any custom properties for your contact.<br />Please [add custom properties](https://loops.so/docs/contacts/properties#custom-contact-properties) in your Loops account before using them with the SDK.<br />Values can be of type `string`, `number`, `nil` (to reset a value), `boolean` or `date` ([see allowed date formats](https://loops.so/docs/contacts/properties#dates)). |
| `mailing_lists` | object | No       | An object of mailing list IDs and boolean subscription statuses.                                                                                                                                                                                                                                                                                                                                                    |

#### Example

```ruby
contact_properties = {
  firstName: "Bob" /* Default property */,
  favoriteColor: "Blue" /* Custom property */,
};
response = LoopsSdk::Contacts.update(
  email: "hello@gmail.com",
  properties: contact_properties
)

# Updating a contact's email address using userId
response = LoopsSdk::Contacts.update(
  email: "newemail@gmail.com",
  properties: {
    userId: "1234",
  })
```

#### Response

This method will return a success or error message:

```json
{
  "success": true,
  "id": "id_of_contact"
}
```

```json
{
  "success": false,
  "message": "An error message here."
}
```

---

### Contacts.find()

Find a contact.

[API Reference](https://loops.so/docs/api-reference/find-contact)

#### Parameters

You must use one parameter in the request.

| Name      | Type   | Required | Notes |
| --------- | ------ | -------- | ----- |
| `email`   | string | No       |       |
| `user_id` | string | No       |       |

#### Examples

```ruby
response = LoopsSdk::Contacts.find(email: "hello@gmail.com")

response = LoopsSdk::Contacts.find(userId: "12345")
```

#### Response

This method will return a list containing a single contact object, which will include all default properties and any custom properties.

If no contact is found, an empty list will be returned.

```json
[
  {
    "id": "cll6b3i8901a9jx0oyktl2m4u",
    "email": "hello@gmail.com",
    "firstName": "Bob",
    "lastName": null,
    "source": "API",
    "subscribed": true,
    "userGroup": "",
    "userId": "12345",
    "mailingLists": {
      "cm06f5v0e45nf0ml5754o9cix": true
    },
    "favoriteColor": "Blue" /* Custom property */
  }
]
```

---

### Contacts.delete()

Delete a contact, either by email address or `userId`.

[API Reference](https://loops.so/docs/api-reference/delete-contact)

#### Parameters

You must use one parameter in the request.

| Name      | Type   | Required | Notes |
| --------- | ------ | -------- | ----- |
| `email`   | string | No       |       |
| `user_id` | string | No       |       |

#### Example

```ruby
response = LoopsSdk::Contacts.delete(email: "hello@gmail.com")

response = LoopsSdk::Contacts.delete(userId: "12345")
```

#### Response

This method will return a success or error message:

```json
{
  "success": true,
  "message": "Contact deleted."
}
```

```json
{
  "success": false,
  "message": "An error message here."
}
```

---

### MailingLists.list()

Get a list of your account's mailing lists. [Read more about mailing lists](https://loops.so/docs/contacts/mailing-lists)

[API Reference](https://loops.so/docs/api-reference/list-mailing-lists)

#### Parameters

None

#### Example

```ruby
response = LoopsSdk::MailingLists.list
```

#### Response

This method will return a list of mailing list objects containing `id`, `name` and `isPublic` attributes.

If your account has no mailing lists, an empty list will be returned.

```json
[
  {
    "id": "cm06f5v0e45nf0ml5754o9cix",
    "name": "Main list",
    "isPublic": true
  },
  {
    "id": "cm16k73gq014h0mmj5b6jdi9r",
    "name": "Investors",
    "isPublic": false
  }
]
```

---

### Events.send()

Send an event to trigger an email in Loops. [Read more about events](https://loops.so/docs/events)

[API Reference](https://loops.so/docs/api-reference/send-event)

#### Parameters

| Name                 | Type   | Required | Notes                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| -------------------- | ------ | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `event_name`         | string | Yes      |                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `email`              | string | No       | The contact's email address. Required if `user_id` is not present.                                                                                                                                                                                                                                                                                                                                                                                            |
| `user_id`            | string | No       | The contact's unique user ID. If you use `user_id` without `email`, this value must have already been added to your contact in Loops. Required if `email` is not present.                                                                                                                                                                                                                                                                                     |
| `contact_properties` | object | No       | An object containing contact properties, which will be updated or added to the contact when the event is received.<br />Please [add custom properties](https://loops.so/docs/contacts/properties#custom-contact-properties) in your Loops account before using them with the SDK.<br />Values can be of type `string`, `number`, `nil` (to reset a value), `boolean` or `date` ([see allowed date formats](https://loops.so/docs/contacts/properties#dates)). |
| `event_properties`   | object | No       | An object containing event properties, which will be made available in emails that are triggered by this event.<br />Values can be of type `string`, `number`, `boolean` or `date` ([see allowed date formats](https://loops.so/docs/events/properties#important-information-about-event-properties)).                                                                                                                                                        |
| `mailing_lists`      | object | No       | An object of mailing list IDs and boolean subscription statuses.                                                                                                                                                                                                                                                                                                                                                                                              |

#### Examples

```ruby
response = LoopsSdk::Events.send(
  event_name: "signup",
  email: "hello@gmail.com"
)

response = LoopsSdk::Events.send(
  event_name: "signup",
  email: "hello@gmail.com",
  event_properties: {
    username: "user1234",
    signupDate: "2024-03-21T10:09:23Z",
  },
  mailing_lists: {
    cm06f5v0e45nf0ml5754o9cix: true,
    cm16k73gq014h0mmj5b6jdi9r: false,
  },
})

# In this case with both email and userId present, the system will look for a contact with either a
#  matching `email` or `user_id` value.
# If a contact is found for one of the values (e.g. `email`), the other value (e.g. `user_id`) will be updated.
# If a contact is not found, a new contact will be created using both `email` and `user_id` values.
# Any values added in `contact_properties` will also be updated on the contact.
response = LoopsSdk::Events.send(
  event_name: "signup",
  email: "hello@gmail.com",
  user_id: "1234567890",
  contact_properties: {
    firstName: "Bob",
    plan: "pro",
  },
})
```

#### Response

This method will return a success or error:

```json
{
  "success": true
}
```

```json
{
  "success": false,
  "message": "An error message here."
}
```

---

### Transactional.send()

Send a transactional email to a contact. [Learn about sending transactional email](https://loops.so/docs/transactional/guide)

[API Reference](https://loops.so/docs/api-reference/send-transactional-email)

#### Parameters

| Name                         | Type     | Required | Notes                                                                                                                                                                                            |
| ---------------------------- | -------- | -------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `transactional_id`           | string   | Yes      | The ID of the transactional email to send.                                                                                                                                                       |
| `email`                      | string   | Yes      | The email address of the recipient.                                                                                                                                                              |
| `add_to_audience`            | boolean  | No       | If `true`, a contact will be created in your audience using the `email` value (if a matching contact doesn’t already exist).                                                                     |
| `data_variables`             | object   | No       | An object containing data as defined by the data variables added to the transactional email template.<br />Values can be of type `string` or `number`.                                           |
| `attachments`                | object[] | No       | A list of attachments objects.<br />**Please note**: Attachments need to be enabled on your account before using them with the API. [Read more](https://loops.so/docs/transactional/attachments) |
| `attachments[].filename`     | string   | No       | The name of the file, shown in email clients.                                                                                                                                                    |
| `attachments[].content_type` | string   | No       | The MIME type of the file.                                                                                                                                                                       |
| `attachments[].data`         | string   | No       | The base64-encoded content of the file.                                                                                                                                                          |

#### Examples

```ruby
response = LoopsSdk::Transactional.send(
  transactional_id: "clfq6dinn000yl70fgwwyp82l",
  email: "hello@gmail.com",
  data_variables: {
    loginUrl: "https://myapp.com/login/",
  },
)

# Please contact us to enable attachments on your account.
response = LoopsSdk::Transactional.send(
  transactional_id: "clfq6dinn000yl70fgwwyp82l",
  email: "hello@gmail.com",
  data_variables: {
    loginUrl: "https://myapp.com/login/",
  },
  attachments: [
    {
      filename: "presentation.pdf",
      content_type: "application/pdf",
      data: "JVBERi0xLjMKJcTl8uXrp/Og0MTGCjQgMCBvYmoKPD...",
    },
  ],
})
```

#### Response

This method will return a success or error message.

```json
{
  "success": true
}
```

If there is a problem with the request, a descriptive error message will be returned:

```json
{
  "success": false,
  "path": "dataVariables",
  "message": "There are required fields for this email. You need to include a 'dataVariables' object with the required fields."
}
```

```json
{
  "success": false,
  "error": {
    "path": "dataVariables",
    "message": "Missing required fields: login_url"
  },
  "transactionalId": "clfq6dinn000yl70fgwwyp82l"
}
```

---

### CustomFields.list()

Get a list of your account's custom fields. These are custom properties that can be added to contacts to store extra data. [Read more about contact properties](https://loops.so/docs/contacts/properties)

[API Reference](https://loops.so/docs/api-reference/list-custom-fields)

#### Parameters

None

#### Example

```ruby
response LoopsSdk::CustomFields.list
```

#### Response

This method will return a list of custom field objects containing `key`, `label` and `type` attributes.

If your account has no custom fields, an empty list will be returned.

```json
[
  {
    "key": "favoriteColor",
    "label": "Favorite Color",
    "type": "string"
  },
  {
    "key": "plan",
    "label": "Plan",
    "type": "string"
  }
]
```

---

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/Loops-so/loops-rb.
