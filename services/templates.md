title: Templates
description: Manage reusable content templates that are run through the SparkPost substitution engine and can be used when sending messages.

# Group Templates

A template is a named collection of content stored on the server side.  Templates are used in a transmission by providing the id of the template at the time of transmission submission.  Each textual component of the
template (headers, text, and html) is run through the substitution engine
to produce recipient specific email messages.  The Templates API provides the means to manage your templates.

## Using Postman

If you use [Postman](https://www.getpostman.com/) you can click the following button to import the SparkPost API as a collection:

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://www.getpostman.com/run-collection/81ee1dd2790d7952b76a)

## Template Attributes

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|id    |string  |Short, unique, alphanumeric ID used to reference the template   | At a minimum, id or name is required upon creation.  It is auto generated if not provided. |After a template has been created, this property cannot be changed.  Maximum length - 64 bytes   |
|content              |JSON  |Content that will be used to construct a message  |  yes  |  For a full description, see the [Content Attributes](#header-content-attributes). Maximum length - 20 MBs  |
|published |boolean |Whether the template is published or is a draft version|no - defaults to false|A template cannot be changed from published to draft.|
|name |string  |Editable display name  | At a minimum, id or name is required upon creation.   |The name does not have to be unique.  Maximum length - 1024 bytes   |
|description |string  |Detailed description of the template  |no    | Maximum length - 1024 bytes |
|options |JSON |JSON object in which template options are defined|no| For a full description, see the [Options Attributes](#header-options-attributes).|
|shared_with_subaccounts | boolean | Whether this template can be used by subaccounts | no | Defaults to `false`.  Only available to templates  belonging to a master account.|
|has_draft | boolean | Whether the template has a draft version | read-only | |
|has_published | boolean | Whether the template has a published version | read-only | |


### Content Attributes

Content for a template is described in a JSON object with the following fields:

| Field         | Type     | Description                           | Required   | Notes   |
|------------------------|:-:       |---------------------------------------|-------------|--------|
|html    |string  |HTML content for the email's `text/html` MIME part|At a minimum, html or text is required.  |Expected in the UTF-8 charset with no `Content-Transfer-Encoding` applied.  Substitution syntax is supported. |
|text    |string  |Text content for the email's `text/plain` MIME part|At a minimum, html or text is required. |Expected in the UTF-8 charset with no `Content-Transfer-Encoding` applied.  Substitution syntax is supported.|
|subject |string  |Email subject line   | yes |Expected in the UTF-8 charset without RFC2047 encoding.  Substitution syntax is supported. |
|from |string or JSON  | Address `"from" : "deals@company.com"` or JSON object composed of the `name` and `email` fields `"from" : { "name" : "My Company", "email" : "deals@company.com" }` used to compose the email's `From` header| yes | Substitution syntax is supported. |
|reply_to |string  |Email address used to compose the email's `Reply-To` header | no | Substitution syntax is supported. |
|headers| JSON | JSON dictionary containing headers other than `Subject`, `From`, `To`, and `Reply-To`  | no | See the [Header Notes](#header-header-notes). |

#### Header Notes

* Headers such as `Content-Type` and `Content-Transfer-Encoding` are not allowed here as they are auto generated upon construction of the email.
* The `To` header should not be specified here, since it is generated from each recipient's `address.name` and `address.email`.
* Each header value is expected in the UTF-8 charset without RFC2047 encoding.
* Substitution syntax is supported.

Alternately, the content JSON object may contain a single `email_rfc822` field. `email_rfc822` is mutually exclusive with all of the above fields.

| Field         | Type     | Description                           | Required   | Notes   |
|--------------------|:-:       |---------------------------------------|-----------------------|--------|
|email_rfc822    |string  |Pre-built message with the format as described by the [message/rfc822 Content-Type](http://tools.ietf.org/html/rfc2046#section-5.2.1) |no   |  See the [email_rfc822 Notes](#header-email_rfc822-notes). |

#### email_rfc822 Notes

* Substitutions will be applied in the top-level headers and the first non-attachment `text/plain` and
first non-attachment `text/html` MIME parts only.
* Lone `LF`s and lone `CR`s are allowed. The system will convert line endings to `CRLF` where
necessary.
* The provided `email_rfc822` should NOT be dot stuffed.  The system dot stuffs before sending the outgoing message.
* The provided `email_rfc822` should NOT contain the SMTP terminator `\r\n.\r\n`.  The system always adds this terminator.
* The provided `email_rfc822` in MIME format will be rejected if SparkPost cannot parse the contents into a MIME tree.

### Options Attributes

Options for a template are described in a JSON object, as follows, using the transmission-level option if not specified:

| Field         | Type     | Description                           | Required   | Notes   |
|--------------------|:-:       |---------------------------------------|-------------|------------------|
|open_tracking |boolean |Enable or disable open tracking | no | To override the default for a specific transmission, specify the `options.open_tracking` field upon creation of the transmission. |
|click_tracking |boolean |Enable or disable click tracking | no | To override the default for a specific transmission, specify the `options.click_tracking` field upon creation of the transmission. |
|transactional |boolean |Distinguish between transactional and non-transactional messages for unsubscribe and suppression purposes | no | To override the default for a specific transmission, specify the `options.transactional` field upon creation of the transmission. |

## Error Attributes

On success, the API returns a `results` JSON object along with HTTP 200.  On failure, an `errors` JSON array will be returned along with HTTP 4xx or 5xx.  Each error is described in a JSON object with the following fields:

| Field         | Type     | Description                           |  Example |
|--------------------|:-:       |---------------------------------------|--------|
|message |string | Explains the class of error  | `"substitution language syntax error in template content"` |
|code |string| Identifies the class of error| `"3000"` |
|description|string| Detailed explanation of error| `"Error while compiling part text: line 4: syntax error near 'age'"` |
|part|string| For substitution errors, identifies the MIME part where the error occurred | `"text"`, `"html"`, `"Header:Subject"`, `"text/plain"` |
|line|number| For substitution errors, identifies the line number within the MIME part identified by the `part` JSON field | `4` |

## Create and List [/templates]

### Create a Template [POST]

Create a template by providing a **template object** as the POST request body.

At a minimum, the `name` and `content` fields are required, where content must contain the `from`, `subject`, and at least one of `html` or `text` fields.

By default, when a template is referenced in a transmission, it is the published version of that template.  To submit a transmission that uses a draft template, set the transmission field `use_draft_template` to true.  For additional details, see the [Transmissions API documentation](transmissions.html) for [Using a Stored Template](transmissions.html#header-using-a-stored-template).


#### Create Parts

The following are key points about creating parts in your templates, as shown in the example:

* The `id` field may be supplied, and it must be unique.
* By default, templates are created as drafts.  If you would like to directly publish a template upon creation, set the `published` field to true.
* Open and click tracking may be enable/disabled at the template level using the `open_tracking` and
`click_tracking` fields.
* The `from` field may be a JSON object composed of `email` and `name`.
* A `Reply-To` header may be specified using the `reply_to` field.
* Both `text` and `html` may be provided.
* Additional headers may be specified in the `headers` JSON dictionary.


#### Create RFC822

Fully formed email_rfc822 content may be provided instead of the `text`, `html`, `from`, and `subject` parts, as shown in the example.



+ Request Create Basic Template (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
            "name" : "Summer Sale!",
            "shared_with_subaccounts": false,

            "content": {
                "from": "marketing@bounces.company.example",
                "subject": "Summer deals",
                "html": "<b>Check out these deals!</b>"
            }
        }
        ```

+ Response 200 (application/json)

        {
          "results": {
            "id": "11806290401558530"
          }
        }

+ Response 400 (application/json)

        {
          "errors" : [
            {
              "description" : "Unconfigured or unverified sending domain.",
              "code" : "7001",
              "message" : "Invalid domain"
            }
          ]
        }

+ Response 400 (application/json)

        {
          "errors" : [
            {
              "description" : "Subaccounts cannot set the shared_with_subaccounts flag",
              "code" : "1200",
              "message" : "invalid params"
            }
          ]
        }

+ Response 422 (application/json)

        {
          "errors" : [
            {
              "part" : "text",
              "description" : "Error while compiling part text: line 4: syntax error near 'age'",
              "line" : 4,
              "code" : "3000",
              "message" : "substitution language syntax error in template content"
            }
          ]
        }

+ Request Create Parts (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

        ```js
        {
          "id" : "summer_sale",
          "name" : "Summer Sale!",
          "published" : true,
          "description": "Template for a Summer Sale!",
          "shared_with_subaccounts": false,
          "options": {
            "open_tracking" : false,
            "click_tracking" : true
          },
          "content": {
            "from": {
              "email": "marketing@bounces.company.example",
              "name": "Example Company Marketing"
            },

            "subject": "Summer deals for {{name}}",
            "reply_to": "Summer deals <summer_deals@company.example>",

            "text": "Check out these deals {{name}}!",
            "html": "<b>Check out these deals {{name}}!</b>",

            "headers": {
              "X-Customer-Campaign-ID": "Summer2014"
            }
          }
        }
        ```

+ Response 200 (application/json)

        {
          "results": {
            "id": "summer_sale"
          }
        }


+ Request Create RFC822 (application/json)

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

        ```js
        {
          "id" : "another_summer_sale",
          "name" : "Summer Sale!",
          "published" : true,

          "options": {
            "open_tracking" : false,
            "click_tracking" : true
          },

          "content": {
            "email_rfc822" : "Content-Type: text/plain\nFrom: Example Company Marketing <marketing@bounces.company.example>\nReply-To:Summer deals <summer_deals@company.example>\nX-Customer-Campaign-ID: Summer2014\nSubject: Summer deals for {{name}}\n\nCheck out these deals {{name}}!"
          }
        }
        ```

+ Response 200 (application/json)

        {
          "results": {
            "id": "another_summer_sale"
          }
        }

### List all Templates [GET /templates]

Lists the most recent version of each template in your account.

Each template object in the list will have the following fields:
- id: Unique template ID.
- name: Template name.
- published: Published state of the template (true = published, false = draft).
- description: Template description.
- last_update_time: The time the template was last updated.
- has_draft: flag indicating whether the template has a draft version.
- has_published: flag indicating whether the template has a published version.

Additional, templates owned by the Master subaccount will have the following field:
- shared_with_subaccounts: Whether the template is shared with subaccounts.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results" : [
            {
              "id" : "summer_sale",
              "name" : "Summer Sale!",
              "published" : true,
              "description" : "",
              "has_draft": true,
              "has_published": true,
              "last_update_time": "2017-08-11T12:12:12+00:00",
              "shared_with_subaccount" : true
            },
            {
              "id" : "daily",
              "name" : "daily",
              "published" : false,
              "description" : "",
              "has_draft": true,
              "has_published": true,
              "last_update_time": "2017-08-10T14:15:16+00:00",
              "shared_with_subaccount" : false

            }
          ]
        }

### List Templates selectively [GET /templates{?draft,shared_with_subaccounts}]

Lists the most recent version of each shared draft template to which you have access.

Each template object in the list will have the following fields:
- id: Unique template ID.
- name: Template name.
- published: Published state of the template (true = published, false = draft).
- description: Template description.
- last_update_time: The time the template was last updated.

Additional, templates owned by the Master subaccount will have the following field:
- shared_with_subaccounts: Whether the template is shared with sub-accounts.

+ Parameters
    + draft (optional, boolean, `true`) ...If true, returns the most recent draft template.  If false, returns the most recent published template.  When not provided, returns the most recent template version regardless of draft or published.
    + shared_with_subaccounts (optional, boolean, `true`) ...If true, returns only shared templates. If false, returns only non-shared templates.  When not provided, returns both shared and non-shared templates.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results" : [
            {
              "id" : "fall_sale",
              "name" : "Fall Sale!",
              "published" : false,
              "description" : "",
              "has_draft": true,
              "has_published": true,
              "last_update_time": "2017-08-10T14:15:16+00:00",
              "shared_with_subaccount" : true
            },
            {
              "id" : "weekly",
              "name" : "weekly",
              "published" : false,
              "description" : "",
              "has_draft": true,
              "has_published": true,
              "last_update_time": "2017-08-08T16:15:26+00:00",
              "shared_with_subaccount" : true
            }
          ]
        }

## Retrieve [/templates/{id}{?draft}]

### Retrieve a Template [GET]

Retrieve a single template by specifying its ID in the URI path. By default, the most recently
updated version is returned. Use the **draft** query parameter to specify a draft or published
template.


The result will include a `last_update_time` field. The `last_update_time` is the time the template was last updated, for both draft and published versions.

If the template was used for message generation, the result will also include a `last_use` field. The `last_use` time represents the last time any version of this template was used (draft or published).

For a master account owned template **only**, the results will include the `shared_with_subaccounts` field reflecting the template's shared status.

+ Parameters
    + id (required, string, `11714265276872`) ... ID of the template
    + draft (optional, boolean, `true`) ...If true, returns the most recent draft template.  If false, returns the most recent published template.  If not provided, returns the most recent template version regardless of draft or published.

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf
            Accept: application/json

+ Response 200 (application/json)

        {
          "results" : {
            "id" : "summer_sale",
            "name" : "Summer Sale!",
            "description" : "",
            "has_draft": true,
            "has_published": true,
            "published" : false,
            "shared_with_subaccounts" : false,
            "last_update_time": "2014-05-22T15:12:59+00:00",
            "last_use": "2014-06-02T08:15:30+00:00",

            "options": {
              "open_tracking" : false,
              "click_tracking" : true,
              "transactional" : false
            },

            "content": {
              "from": {
                "email": "marketing@bounces.company.example",
                "name": "Example Company Marketing"
              },

              "subject": "Summer deals for {{name}}",
              "reply_to": "Summer deals <summer_deals@company.example>",

              "text": "Check out these deals {{name}}!",
              "html": "<b>Check out these deals {{name}}!</b>",

              "headers": {
                "X-Customer-Campaign-ID": "Summer2014"
              }
            }
          }
        }

## Update [/templates/{id}{?update_published}]

### Update a Template [PUT]

Update an existing template by specifying its ID in the URI path and a **template object** as the PUT request body.
By default, an update operates on an existing draft version.
An existing published version can be overwritten directly by setting the `update_published` query parameter to `true`.

<div class="alert alert-info"><strong>Note</strong>: Attempting to update the published version of a template when only a draft version exists (and vice versa) will result in an error.</div>

The `name` field may be modified, but the `id` field is read only.

If a content object is provided in the update request, it must
contain all relevant content fields whether they are being changed or not.
The new content will completely overwrite the existing content.

The example shows an update that will rename the template, enable open tracking,
and update the content all in one API call. All content fields are included whether they are being
changed or not.

Publishing a template is a special kind of update.
It uses the most recent draft version to create a new published version, even if one does not already exist.
The body of the PUT request should contain the `"published": true` field as shown in the second example below (`Publish`).

+ Parameters
    + id (required, string, `11714265276872`) ... ID of the template
    + update_published = `false` (optional, boolean, `false`) ...If true, directly overwrite the existing published template.  If false, update the existing draft.

+ Request Update (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
          "options" : {
            "open_tracking": true
          },
          "name" : "A new name!",
          "shared_with_subaccounts": true,
          "content": {
            "from": {
              "email": "marketing@bounces.company.example",
              "name": "Example Company Marketing"
            },
            "subject": "Updated Summer deals for {{name}}",
            "reply_to": "Summer deals <summer_deals@company.example>",
            "text": "Updated: Check out these deals {{name}}!",
            "html": "<b>Updated: Check out these deals {{name}}!</b>"
          }
        }
        ```

+ Response 200


+ Request Publish (application/json)

  When publishing a draft template, `update_published` should be omitted, or specified as `false`.

  + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

  + Body

        ```js
        {
          "published": true
        }
        ```

+ Response 200


## Preview [/templates/{id}/preview{?draft}]

### Preview a Template [POST]

Preview the most recent version of an existing template by specifying `{id}/preview` in the URI path
and providing `substitution_data` as part of the POST request body.
The template's content will be expanded using the substitution data provided and returned
in the response. By default, the most recently updated version is returned.  Use the `draft` query parameter to specify a draft or published
template.

See the [Substitutions Reference section](substitutions-reference.html) for more information.

+ Parameters
    + id (required, string, `11714265276872`) ... ID of the template
    + draft (optional, boolean, `true`) ...If true, previews the most recent draft template.  If false, previews the most recent published template.  If not provided, previews the most recent template version regardless of draft or published.

+ Request (application/json)

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

    + Body

        ```
        {
          "substitution_data" : {
            "name" : "Natalie",
            "age" : 35,
            "member" : true
          }
        }
        ```

+ Response 200 (application/json)

        {
            "results" : {
                "from": {
                    "email": "marketing@bounces.company.example",
                    "name": "Example Company Marketing"
                },
                "subject": "Summer deals for Natalie",
                "reply_to": "Summer deals <summer_deals@company.example>",
                "text": "Check out these deals Natalie!",
                "html": "<b>Check out these deals Natalie!</b>",
                "headers": {
                    "X-Customer-Campaign-ID": "Summer2014"
                }
            }
        }

## Delete [/templates/{id}]

### Delete a Template [DELETE]

Delete a template by specifying its ID in the URI path.
If the template delete API call succeeds, then ALL versions of the template will be deleted from the system (both published AND draft versions).

<div class="alert alert-info">If a transmission uses a stored template, the template cannot be deleted if the transmission is submitted or generating.</div>

+ Parameters
    + id (required, string, `11714265276872`) ... ID of the template

+ Request

    + Headers

            Authorization: 14ac5499cfdd2bb2859e4476d2e5b1d2bad079bf

+ Response 200 (application/json)

        {
        }

+ Response 404 (application/json)

    + Body

        ```
        {
        "errors": [
          {
            "message": "resource not found",
            "code": "1600",
            "description": "Template does not exist"
          }
        ]
        }
        ```

+ Response 409 (application/json)

    + Body

        ```
        {
        "errors": [
          {
            "message": "resource conflict",
            "code": "1602",
            "description": "Template is in use by msg generation"
          }
        ]
        }
        ```
