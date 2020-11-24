# API Spec for Upwork Job

Last updated: 2020-11-24

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

## Fetching User Data

There is only one piece of JSON that needs to be fetched. This is:

`HTTP GET /api/data` should return `200` Status Code with JSON like:

```json
{
    "user": {
        "phoneNumber": "07777777777",
        "firstName": "Alice",
        "lastName": "Smith",
        "estimatedDueDate": "2020-01-01T00:00:00",
        "placeOfBirth": "LocationX",
        "birthPartnersArray": [],
        "expecting": "A Boy"
    },
    "birthPlans": [
        {
            "id": 6,
            "displayOrder": 0,
            "name": "Secondary Plan",
            "title": "This is some other custom text for the secondary plan",
            "layoutItems": [
                {
                    "x": 1,
                    "y": 1,
                    "width": 3,
                    "height": 2,
                    "imageId": 1,
                    "itemText": "This is where things should start"
                },
                {
                    "x": 5,
                    "y": 5,
                    "width": 3,
                    "height": 2,
                    "imageId": 25,
                    "itemText": "One before the very end"
                },
                {
                    "x": 5,
                    "y": 5,
                    "width": 3,
                    "height": 2,
                    "imageId": 0,
                    "itemText": "This is just a very long message. And no picture"
                }
            ],
            "items": []
        },
        {
            "id": 7,
            "displayOrder": 0,
            "name": "Main",
            "title": "This is how I really want things to go",
            "layoutItems": [
                {
                    "x": 1,
                    "y": 1,
                    "width": 3,
                    "height": 2,
                    "imageId": 5,
                    "itemText": "The second vowel of the alphabet"
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
    ],
    "readOnlyShares": [
        {
            "id": 1,
            "sharedWithUserId": 3,
            "sharedWithPhoneNumber": "07999999999",
            "sharedWithUserName": "Charlie",
            "accessRight": 1
        }
    ],
    "otherBirthPlans": []
}
```

If the Status Code is not `200`, an error message should be show.

## Updating User Data

Changes to any of the `Data.User` fields, should saved by doing `HTTP POST /api/user/` with the `User` object as the body. e.g:

```json
{
    "phoneNumber": "07777777777",
    "firstName": "Alice",
    "lastName": "Smith",
    "estimatedDueDate": "2020-01-01",
    "placeOfBirth": "LocationX",
    "birthPartnersArray": [],
    "expecting": "A Boy"
}
```

The result should be a `200` Status Code. If not, an error message should be shown.

After updating the data, it is important to refresh the `Data` (see Fetching User Data section above)

# Inserting/Updating/Deleting Birth Plan Data

### Inserting BirthPlan Data

`HTTP POST /api/plan` with JSON like:

```json
{
    "Name": "Main",
    "Title": "This is how I really want things to go",
	"Items": [],
    "LayoutItems":[
        {
            "X": 1,
            "Y": 1,
            "Width": 3,
            "Height": 2,
            "ImageId": 5,
            "ItemText": "The second vowel of the alphabet"
        },
        {
            "X": 4,
            "Y": 1,
            "Width": 3,
            "Height": 2,
            "ImageId": 26,
            "ItemText": "The end of the road for me!"
        }
    ]
}
```

### Updating BirthPlan Data

`HTTP PUT /api/plan/{Id}` with JSON like (important: with Id field):

```json
{
    "id": 7,
    "displayOrder": 0,
    "name": "A New Name!",
    "title": "This is how I really want things to go actually!",
    "layoutItems": [
        {
            "x": 1,
            "y": 1,
            "width": 3,
            "height": 2,
            "imageId": 5,
            "itemText": "The second vowel of the alphabet!"
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

### Deleting BirthPlan Data

`HTTP DELETE /api/plan/{Id}` with no body

## Sharing With Others

### To Share

`HTTP POST /api/share` with JSON including details of the person to share with:

```json
{
    "PhoneNumber": "07999999999",
    "UserName": "Charlie"
}
```

### To Unshare

`HTTP DELETE /api/share/{id}` where `id` is from `Data.ReadOnlyShares[index].Id`

## Fetching ReferenceData

All reference data can be fetched from `HTTP GET /api/refdata/` which returns a JSON like:

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
