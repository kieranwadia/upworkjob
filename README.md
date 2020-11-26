# API Spec for Upwork Job

Last updated: 2020-11-26

This spec lists all the endpoints in the API and how to use them.

## Logging In / Authentication

The user is considered authenticated if they have a certain cookie on their system. The cookie will be created automatically by the browser in response to a valid login step (see below).

### Checking Whether User Is Logged In

Call `HTTP GET /api/user`:

*  It will return `401` Status Code if user is not logged in
* It  will return `200` Status Code and a Json of a `User` if the user is logged in:


```json
{
  "id": 0,
  "phoneNumber": "07777777777",
  "firstName": "Alice",
  "lastName": "Smith",
  "estimatedDueDate": "2020-01-01",
  "placeOfBirth": "Kings College Hospital",
  "birthPartnersArray": [],
  "expecting": "A Baby Boy"
}
```

### Logging In: Step 1 of 2: Request Passcode To Be Sent (to Phone)

Call `HTTP POST /api/passcode` with the user's phone number:

```json
{
	"PhoneNumber": "07777777777"
}
```

It should return `200` Status Code. In the background, this sends a text to the user's mobile with with a 4-digit passcode.

If the Status Code is not `200`, an error message should be show.

### Logging In: Step 2 of 2: Providing The Passcode To Be Sent

Call `HTTP POST /api/login` with data:

```json
{
	"PhoneNumber": "07777777777",
    "Passcode": 5555
}
```

It should return `200` Status Code with data like:

```json
{
    "success": false,
    "authId": null,
    "failureReason": "Hmm... the Passcode isn't right; check and try again!",
    "requestPasscodeRetry": true,
    "requestPhoneNumberResend": false
}
```

* If `Success` is true, the user has been logged in (a cookie will be created automatically).
* If `Success` is false and `requestPasscodeRetry` is true, request the user to re-enter the Passcode.
* If `Success` is false and `requestPhoneNumberResend` is true, request the user to re-enter the PhoneNumber.

If the Status Code is not `200`, an error message should be show.

### Logging Out

To log out at any point (assuming the user is already logged in)

`HTTP DELETE /api/login`

## User Resource

Resource URI: `/api/user`  (note that no 'Id' is required because the user is derived from Auth code in the cookie)

`HTTP GET` 

* Will return the logged in user

`HTTP PUT` 

  * Amend user

#### Example `User` resource

```json
{
    "phoneNumber": "07777777777",
    "firstName": "Alice",
    "lastName": "Smith",
    "estimatedDueDate": "2020-01-01T00:00:00",
    "placeOfBirth": "LocationX",
    "birthPartnersArray": [
        {
            "relation": "Partner",
            "name": "John"
        },
        {
            "relation": "Doula",
            "name": "Jane"
        }
    ],
    "expecting": "A Boy"
}
```

If the Status Code is not `200`, an error message should be show.

## BirthPlan Resource

Resource URI: `/api/birthplan` 

`HTTP GET` 

* Will return an array of `BirthPlan ` (owned by the logged in user)

`HTTP GET /api/birthplan/{id}` 

  * Will return a single `BirthPlan` for the given id (or a `404` if not found)

`HTTP POST /api/birthplan` 

  * To create a new `BirthPlan`. The created `BirthPlan` will be returned

`HTTP PUT /api/birthplan/{id}` 

  * To update an existing `BirthPlan`. The updated `BirthPlan` will be returned (or a `404` if not found)

`HTTP DELETE /api/birthplan/{id}` 

  * To delete an existing `BirthPlan` (will return `404` if not found)

#### Example `BirthPlan` resource

```json
{
    "id": 8,
    "displayOrder": 0,
    "name": "Secondary Updated Plan",
    "title": "If things don't go as planned, I'm happy with this",
    "layoutItems": [
        {
            "x": 1,
            "y": 1,
            "width": 3,
            "height": 2,
            "imageId": 5,
            "itemText": "The second vowel of the alphabet..."
        },
        {
            "x": 4,
            "y": 1,
            "width": 3,
            "height": 2,
            "imageId": 26,
            "itemText": "The end of the road for me!"
        }
    ],
    "items": []
}
```

## Share Resource

This is a list of people who the user has shared their BirthPlans with.

Resource URI: `/api/share` 

`HTTP GET` 

* Will return an array of `Share`s (owned by the logged in user)

`HTTP POST /api/share` 

  * To create a new `Share`. Only requires two fields:  `{"PhoneNumber": "07999999999","UserName": "Charlie"}` . Will return the newly created `Share`.

`HTTP DELETE /api/share/{shareId}` 

  * To delete a Share (will return `404` if not found)

### Example Share[] resource

```json
[
    {
        "id": 1,
        "sharedWithUserId": 3,
        "sharedWithPhoneNumber": "07999999999",
        "sharedWithUserName": "Charlie"
    },
    {
        "id": 2,
        "sharedWithUserId": 5,
        "sharedWithPhoneNumber": "07888888888"
    },
]
```

## OtherBirthPlans Resource

This is a list of `BirthPlan`s that other people have shared with the current user. The current user can only view them (no editing allowed).

Resource URI: `/api/otherbirthplans` 

* Will return an array of `OtherBirthPlans`s (for the logged in user)

`HTTP GET /api/otherbirthplans` 

```json
[
    {
        "shareId": 1,
        "sharedByUserId": 2,
        "sharedByUserName": "Alice",
        "birthPlans": [
            {
                "id": 8,
                "displayOrder": 0,
                "name": "Secondary Updated Plan",
                "title": "If things don't go as planned, I'm happy with this",
                "layoutItems": [
                    {
                        "x": 4,
                        "y": 1,
                        "width": 3,
                        "height": 2,
                        "imageId": 26,
                        "itemText": "The end of the road for me!"
                    },
                    {
                        "x": 1,
                        "y": 1,
                        "width": 3,
                        "height": 2,
                        "imageId": 5,
                        "itemText": "The second vowel of the alphabet..."
                    }
                ],
                "items": []
            },
            {
                "id": 9,
                "displayOrder": 0,
                "name": "Thirdly",
                "title": "If things don't go as planned, I'm happy with this",
                "layoutItems": [
                    {
                        "x": 1,
                        "y": 1,
                        "width": 3,
                        "height": 2,
                        "imageId": 5,
                        "itemText": "The second vowel of the alphabet"
                    }
                ],
                "items": []
            }
        ]
    }
]
```

## Fetching ReferenceData

All reference data can be fetched from `HTTP GET /api/refdata` which returns a JSON like:

```json
{
    "images": [
        {
            "id": 1,
            "fileName": "A.PNG",
            "imageName": "TheLetterA",
            "keywords": [
                "Vowel",
                "Aye"
            ],
            "defaultDescription": "The Letter A is number 1 in the alphabet"
        },
        {
            "id": 2,
            "fileName": "B.PNG",
            "imageName": "TheLetterB",
            "keywords": [
                "Constanant",
                "Bee"
            ],
            "defaultDescription": "The Letter B is number 2 in the alphabet"
        }...
    ],
    "placeOfBirths": [
        "Kings College Hospital",
        "UCHL",
        ...
    ],
    "expectings": [
        "A Boy",
        "A Girl",
        ...
    ],
    "birthPartnerRelationships": [
        "Friend",
        "Husband",
        ...
    ]
}
```

