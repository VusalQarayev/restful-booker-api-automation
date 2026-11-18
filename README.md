# Restful Booker API Automation

An end-to-end REST API test automation project built with **Postman**,
**JavaScript**, **Newman**, and **GitHub Actions**. The suite targets
the public [Restful Booker API](https://restful-booker.herokuapp.com/)
and demonstrates API test design, stateful request chaining, contract
validation, negative testing, and CI reporting.

## Project highlights

-   **End-to-end CRUD workflow:** Authenticates, creates a booking,
    retrieves it, updates it with `PUT` and `PATCH`, deletes it, and
    verifies that it can no longer be retrieved.
-   **Dynamic test data:** Uses Postman's Faker library to generate
    booking data and avoid relying on fixed guest details.
-   **JSON Schema validation:** Checks response structure, required
    properties, primitive data types, and date formats.
-   **Reusable quality checks:** Applies collection-level assertions for
    response-time limits, gateway errors, and required environment
    configuration.
-   **Defensive response handling:** Checks status codes and content
    types before parsing JSON, producing clearer failures when the API
    returns plain-text or unexpected responses.
-   **Negative and boundary coverage:** Exercises invalid credentials,
    missing authentication, unknown booking IDs, unsupported `Accept`
    headers, and incomplete booking payloads.
-   **CI execution and reporting:** Runs the suite through GitHub
    Actions and publishes an HTML report as a workflow artifact.
-   **Credential hygiene:** Keeps the environment password blank in the
    committed export and supplies it at runtime.

## Technology stack

  -----------------------------------------------------------------------
  Technology                          Purpose
  ----------------------------------- -----------------------------------
  Postman Collection v2.1             API requests, pre-request scripts,
                                      and test assertions

  JavaScript / Chai                   Test logic and assertions

  JSON Schema / Ajv                   Response contract validation

  Newman                              Command-line collection execution

  `newman-reporter-htmlextra`         HTML test reporting

  GitHub Actions                      Continuous integration

  Node.js and npm                     Dependency management and test
                                      execution
  -----------------------------------------------------------------------

## Repository structure

``` text
restful-booker-api-automation/
├── .github/
│   └── workflows/
│       └── api-tests.yml
├── collections/
│   └── restful-booker.postman_collection.json
├── environments/
│   └── restful-booker-qa.postman_environment.json
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

The `reports/` directory is generated when tests run and should remain
excluded from Git.

## Test coverage

The collection is organized into four folders.

  -----------------------------------------------------------------------------------------------------------------
  Folder                         Request           Method                                      Main checks
  ------------------------------ ----------------- ------------------------------------------- --------------------
  `01_Health_and_Auth`           Health Check -    `GET /ping`                                 Confirms API
                                 Ping                                                          availability; checks
                                                                                               the observed
                                                                                               `201 Created`
                                                                                               plain-text response.

  `01_Health_and_Auth`           Create Auth       `POST /auth`                                Checks the response
                                 Token - Valid                                                 and token format;
                                 Credentials                                                   stores `auth_token`.

  `02_Booking_E2E_CRUD_Flow`     Create Booking -  `POST /booking`                             Generates booking
                                 Valid Payload                                                 data, validates the
                                                                                               response schema,
                                                                                               checks returned
                                                                                               values, and stores
                                                                                               `bookingid`.

  `02_Booking_E2E_CRUD_Flow`     Get Booking by ID `GET /booking/{bookingid}`                  Checks
                                                                                               read-after-write
                                                                                               persistence,
                                                                                               response schema, and
                                                                                               checkout/check-in
                                                                                               date ordering.

  `02_Booking_E2E_CRUD_Flow`     Update Booking -  `PUT /booking/{bookingid}`                  Authenticates with
                                 Full (PUT)                                                    the session cookie
                                                                                               and validates the
                                                                                               replacement booking.

  `02_Booking_E2E_CRUD_Flow`     Partial Update    `PATCH /booking/{bookingid}`                Checks changed
                                 Booking (PATCH)                                               fields and verifies
                                                                                               that omitted fields
                                                                                               remain unchanged.

  `02_Booking_E2E_CRUD_Flow`     Delete Booking by `DELETE /booking/{bookingid}`               Authenticates with
                                 ID                                                            the session cookie
                                                                                               and checks the API's
                                                                                               observed deletion
                                                                                               response.

  `02_Booking_E2E_CRUD_Flow`     Verify Booking    `GET /booking/{bookingid}`                  Confirms that the
                                 Deleted (404)                                                 deleted booking can
                                                                                               no longer be
                                                                                               retrieved.

  `03_Search_and_Filtering`      Filter Bookings   `GET /booking?firstname=...&lastname=...`   Checks URL-encoded
                                 by Guest Name                                                 query parameters,
                                                                                               array response
                                                                                               structure, and
                                                                                               booking ID lookup.

  `04_Negative_and_Edge_Cases`   Create Auth       `POST /auth`                                Checks the
                                 Token - Invalid                                               `Bad credentials`
                                 Credentials                                                   response and
                                                                                               confirms that no
                                                                                               token is returned.

  `04_Negative_and_Edge_Cases`   Update Booking -  `PUT /booking/1`                            Checks the observed
                                 Unauthorized                                                  `403 Forbidden`
                                 (Missing Token)                                               response when the
                                                                                               session cookie is
                                                                                               missing.

  `04_Negative_and_Edge_Cases`   Get Booking -     `GET /booking/99999999`                     Checks the not-found
                                 Non-Existent ID                                               response for an
                                 (404)                                                         unknown booking ID.

  `04_Negative_and_Edge_Cases`   Get Booking -     `GET /booking/1`                            Checks the observed
                                 Unsupported                                                   response to an
                                 Accept Header                                                 unsupported
                                 (418)                                                         `Accept: text/xml`
                                                                                               header.

  `04_Negative_and_Edge_Cases`   Create Booking -  `POST /booking`                             Documents and checks
                                 Missing Required                                              the API's observed
                                 Fields                                                        server error for an
                                                                                               incomplete payload.
  -----------------------------------------------------------------------------------------------------------------

> **Execution order matters:** The CRUD flow uses runtime variables such
> as `auth_token` and `bookingid`. Run the collection sequentially so
> that values created by earlier requests are available to later
> requests.

## API behaviors and observed deviations

The suite records several behaviors observed in the target public
sandbox. These are useful regression checks, but they should not be
interpreted as recommended REST API design.

  ---------------------------------------------------------------------------------------------------------
  Endpoint / scenario      Observed behavior in this        Common REST expectation   Test-suite treatment
                           project                                                    
  ------------------------ -------------------------------- ------------------------- ---------------------
  `GET /ping`              `201 Created` with plain-text    A health check commonly   Asserts the observed
                           body `Created`                   returns `200 OK`          status and body
                                                                                      rather than assuming
                                                                                      JSON.

  `DELETE /booking/{id}`   `201 Created` with plain-text    Commonly `200 OK` or      Checks the behavior
                           body `Created`                   `204 No Content`          currently observed by
                                                                                      the suite.

  `POST /auth` with        `200 OK` with                    Commonly                  Checks the error
  invalid credentials      `{"reason":"Bad credentials"}`   `401 Unauthorized`        message and verifies
                                                                                      that no token is
                                                                                      exposed.

  `POST /booking` with     `500 Internal Server Error`      Commonly                  Includes a negative
  required fields missing                                   `400 Bad Request` with    test to document the
                                                            validation details        observed server-side
                                                                                      failure.

  Date filtering with      Exact check-in boundary may be   Inclusive boundary        Uses guest-name
  `checkin` / `checkout`   excluded by a strict             behavior may be expected, filtering for the
                           greater-than comparison          depending on the API      reliable lookup flow
                                                            contract                  and documents the
                                                                                      date-filter
                                                                                      observation.

  Authenticated booking    Requires                         APIs often support an     Uses the session
  updates/deletes          `Cookie: token={token}`; a       `Authorization: Bearer`   cookie and includes a
                           bearer token is not accepted by  header                    missing-cookie
                           the tested flow                                            negative test.

  `PUT` / `PATCH` for a    May return                       Commonly `404 Not Found`  Defensive status and
  missing or reset ID      `405 Method Not Allowed`         for an unknown resource   content-type checks
                                                                                      help report
                                                                                      unexpected responses.

  Unsupported `Accept`     `Accept: text/xml` may produce   Commonly                  Includes a negative
  header                   `418 I'm a teapot`               `406 Not Acceptable`      content-negotiation
                                                                                      test.
  ---------------------------------------------------------------------------------------------------------

The public sandbox may reset its data and invalidate session tokens
periodically. The suite therefore creates its own booking and token at
runtime rather than depending on persistent test data.

## Getting started

### Prerequisites

-   Node.js 18 or later
-   npm
-   Git
-   Postman (optional, for running the collection in the Postman
    application)

### 1. Clone the repository

``` bash
git clone https://github.com/VusalQarayev/restful-booker-api-automation.git
cd restful-booker-api-automation
```

### 2. Install dependencies

``` bash
npm install
```

Commit `package-lock.json` to the repository. It allows CI to install
the same dependency versions reproducibly and is required when GitHub
Actions uses npm dependency caching.

### 3. Run the tests

``` bash
npm test
```

The npm script supplies the public sandbox password at runtime and
generates an HTML report at:

``` text
reports/newman-report.html
```

Open that file in a browser to review request details, response
payloads, execution times, and assertion results.

### Run the collection directly in Postman

1.  Open Postman and select **Import**.
2.  Import `collections/restful-booker.postman_collection.json`.
3.  Import `environments/restful-booker-qa.postman_environment.json`.
4.  Select the **Restful-Booker-QA** environment.
5.  If required, set the `password` environment variable to the sandbox
    password (`password123`) in your local Postman environment. Do not
    commit populated credentials.
6.  Open the collection and run all folders sequentially using
    Collection Runner.

## GitHub Actions CI

The workflow is located at `.github/workflows/api-tests.yml`.

According to the workflow configuration, the suite is triggered by:

-   Pushes to `main`
-   Pull requests targeting `main`
-   A scheduled daily run at 08:00 UTC
-   Manual execution through **Actions → Restful Booker API Automation
    CI → Run workflow**

After a run completes, open the workflow run in the **Actions** tab and
download the HTML report from its **Artifacts** section. The artifact
name is configured in the workflow and may include the run number.

### Optional: configure a repository secret

If you want to override the default sandbox password:

1.  Open the repository's **Settings → Secrets and variables →
    Actions**.
2.  Select **New repository secret**.
3.  Set the name to `API_PASSWORD`.
4.  Enter the password as the secret value.
5.  Confirm that the workflow reads `API_PASSWORD` and passes it to
    Newman at runtime.

Do not place real credentials in the collection, environment export,
README, or committed workflow file.

## Notes and limitations

-   This project targets a public demo API, not a production service.
    Its data and behavior may change without notice.
-   The API deviations above are observations from the test environment
    and may change over time.
-   A test that asserts an observed API defect is documenting current
    behavior; it does not mean that behavior is correct.
-   The test suite depends on sequential execution because requests
    share runtime variables.
-   The HTML report is generated locally or in CI and should not be
    committed unless there is a specific reason to version a report.

By Vusal Qarayev