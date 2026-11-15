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
- A vehicle that is still parked cannot be deleted.
// These are the features of our production.