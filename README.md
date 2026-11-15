# ParkFlow: Parking lot API

## A RESTful Parking Lot Management API built with Node.js, Express.js(ES Modules) and a JSON file for storage, supporting parking space management, vehicle parking, unparking and fee calculation.


## Tech Stack
// The Tech Stack we used

**Runtime:** Node.js
**Framework:** Express.js 5
**Environment Configuration:** dotenv (It manages configuration settings and server port)
**Logging Request:** Morgan (It is the HTTP logger middleware for monitoring and debugging API)
**Storage:** Json file storage //These are the Tech Stack we used in this project


## Installation and Running
1. **Clone the repository:**
git clone
https://github.com/sumailaaliyu/parkflow-backend.git
cd parkflow-backend
2. **Install the Packages:**
npm install
3. **Create a file named `config.env` in the project root (it is ignored by Git):
PORT=3000(The server runs by default)
NODE_ENV=development
// If you skip this step the server still runs on port 3000.
4. Start the server
npm start
5. You should see `Sever started on port 3000`. The API is at `http://localhost:3000/api/v1/parking-lot`
NODE_ENV=production (It hides error stack traces from the API responses. It leave it as a development` while testing.)


## Features
- Park a vehicle into a numbered parking space
- Unpark a vehicle and calculate the parking fee (bonus feature)
- Prevents the same vehicle from being parked twice, and a space from beung used twice.
- Validates all input and returns clear error messages.
- Unique ID, `createdAt` and `updatedAt` on every record.
- A vehicle that is still parked cannot be deleted 
// These are the features of our prodcution.


## Endpoints

Base URL: `http://localhost:3000/api/v1/parking-lot`

| Method | URL | Description | Success |
|--------|-----|-------------|---------|
| GET | `/` | List all vehicles that are currently parked | 200 |
| GET | `/:id` | Get one vehicle record by id | 200 |
| POST | `/` | Park a new vehicle | 201 |
| PATCH | `/:id` | Unpark a vehicle (body: `{ "status": "unparked" }`) and get the fee | 200 |
| DELETE | `/:id` | Delete a record (only if the vehicle is unparked) | 200 |
//These are the Endspoints

### Request body for parking a vehicle (POST)

| Field | Type | Required | Rules |
|-------|------|----------|-------|
| `numberPlate` | string | yes | Not empty. Saved in uppercase. Must not already be parked |
| `modelNumber` | string | yes | Not empty |
| `parkingNumber` | integer | yes | 1 or more. Must not be in use by another parked vehicle |

### Example: park a vehicle

`POST /api/v1/parking-lot`

```json
{
  "numberPlate": "ABC123XY",
  "modelNumber": "2024",
  "parkingNumber": 1
}
```

Response `201 Created`:

```json
{
  "status": "success",
  "data": {
    "newVehicle": {
      "id": "4e959fe9c0f44ba4a6a57cbb9bc5cbfc",
      "numberPlate": "ABC123XY",
      "modelNumber": "2024",
      "parkingNumber": 1,
      "status": "parked",
      "fee": 0,
      "createdAt": "2026-11-14T19:30:00.000Z",
      "updatedAt": "2026-11-14T19:30:00.000Z"
    }
  }
}
```

### Example: list parked vehicles

`GET /api/v1/parking-lot` returns `200 OK`:

```json
{
  "status": "success",
  "total": 1,
  "data": { "activeParkingLot": [ { "id": "4e959fe9...", "numberPlate": "ABC123XY", "status": "parked" } ] }
}
```

### Example: unpark a vehicle

`PATCH /api/v1/parking-lot/4e959fe9c0f44ba4a6a57cbb9bc5cbfc`

```json
{ "status": "unparked" }
```

Response `200 OK` (fee for less than one hour is 500):

```json
{
  "status": "success",
  "data": {
    "modifiedData": {
      "id": "4e959fe9c0f44ba4a6a57cbb9bc5cbfc",
      "numberPlate": "ABC123XY",
      "status": "unparked",
      "fee": 500,
      "updatedAt": "2026-11-14T19:45:00.000Z"
    }
  }
}
```


### Fee calculation

The fee is **₦500 per hour**. Any part of an hour counts as a full hour. Time is counted from `createdAt` (when the vehicle was parked) until the moment it is unparked.

| Time parked | Hours charged | Fee |
|-------------|---------------|-----|
| 5 minutes | 1 | 500 |
| 61 minutes | 2 | 1000 |
| 3 hours 10 minutes | 4 | 2000 |


### Status codes

| Code | Meaning | Used when |
|------|---------|-----------|
| 200 | OK | A request succeeded |
| 201 | Created | A vehicle was parked |
| 400 | Bad Request | Invalid input, vehicle already parked, space already in use, or deleting a parked vehicle |
| 404 | Not Found | The record or the route does not exist |
| 409 | Conflict | The vehicle has already been unparked |
| 500 | Internal Server Error | Unexpected server or file error |


### Error response format

```json
{
  "status": "fail",
  "message": "Vehicle is already parked"
}
```

In development mode the response also contains an `error` field with the stack trace.


## Project Structure

```
parkflow-backend/
├── data/
│   └── vehicles.json            # stored records (JSON "database")
├── src/
│   ├── controllers/             # request handlers and global error handler
│   ├── routes/                  # URL paths
│   ├── services/                # checks before parking (duplicates)
│   ├── utils/                   # file storage, validator, fee calculator, helpers
│   ├── app.js                   # Express app and middleware
│   └── index.js                 # starts the server
├── config.env                   # your settings (not committed)
└── package.json
```


## Testing with Postman

1. Put `[]` in `data/vehicles.json` so you start from an empty lot, then start the server with `npm start`.
2. In Postman choose **Import** and select `postman/Parkflow.postman_collection.json`.
3. The collection has a variable `baseUrl` set to `http://localhost:3000`. Change it if you use another port.
4. Open the collection and click **Run**, or send the requests one by one **in order**, because later requests use the ids saved by earlier ones.
5. Each request has an automatic test. Green PASS means the status code matched.

| # | Test | Expected |
|---|------|----------|
| 1 | List parked vehicles (empty) | 200, `total: 0` |
| 2 | Park vehicle A (slot 1) | 201 |
| 3 | Park vehicle B (slot 2) | 201 |
| 4 | List parked vehicles | 200, `total: 2` |
| 5 | Get vehicle A by id | 200 |
| 6 | Get unknown id | 404 |
| 7 | Park with an empty body | 400 |
| 8 | Park with a number as the plate | 400 |
| 9 | Park with `parkingNumber` as text | 400 |
| 10 | Park with `parkingNumber` 0 | 400 |
| 11 | Park the same plate again | 400 |
| 12 | Park into an occupied space | 400 |
| 13 | Unpark with a wrong status value | 400 |
| 14 | Unpark an unknown id | 404 |
| 15 | Delete vehicle A while parked | 400 |
| 16 | Unpark vehicle A | 200, `status: unparked`, `fee: 500` |
| 17 | Unpark vehicle A again | 409 |
| 18 | List parked vehicles | 200, `total: 1` (vehicle B is still there) |
| 19 | Delete vehicle A | 200 |
| 20 | Get vehicle A again | 404 |
| 21 | Get vehicle B | 200 (it was not deleted by mistake) |
| 22 | Unknown route | 404 |
| 23 | Malformed JSON body | 400 |

To test the fee for several hours, open `data/vehicles.json`, change the `createdAt` of a parked vehicle to a time about 3 hours earlier (keep the `Z` at the end), save, then unpark it. A 3-hour-10-minute stay gives `fee: 2000`.


## Team Members

| Name | Role | What they built |
|------|------|-----------------|
| Sumaila Aliyu | Project setup and file storage | Folder structure, `fileStorage.js`, data file |
| Sweeney | Routes | `vehicleRoutes.js`, `app.js` |
| Samuel Niyi | Controllers | `parkingLotRouteHandlers.js` |
| Ugochukwu Uwakwe | Helper functions | `reuseableFunctions.js` |
| Vincent Shalom | Validation, services, error handling and fee | `validator.js`, `vehicleService.js`, `AppError.js`, `index.js`, `feeCalculator.js` |
| Sanni Rihanot Oluwatimileyin | Testing and README | Postman collection, `README.md` |


## Known Issues and Limitations

- Data is stored in a JSON file, so it is not suitable for many users writing at the same time.
- There is no authentication; anyone who can reach the API can use it.
- The parking space number is chosen by the client. There is no maximum number of spaces.
- The fee is a flat rate of ₦500 per hour for every vehicle type.
- `GET /` only returns vehicles that are currently parked. Use `GET /:id` to read an unparked record.
- Only the status can be changed (to `unparked`). Plate and model cannot be edited.