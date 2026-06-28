# From Zero to Liquit REST API Results: How GitHub Copilot Helped Me Build My First OAuth2 Script

I am not a REST API specialist, and OAuth2 was not something I was comfortable with when I started this task. I just needed one practical outcome: connect to my Liquit environment and retrieve zone and license information in a repeatable way.

This is the story of how I used GitHub Copilot to get there.

## Where It Started

The first challenge was simple but critical: where do I even begin?

I started by exploring Liquit client resources and found:

- https://api.liquit.com/workspace/v2/liquit.workspace.js

That file became my entry point. With Copilot, I could quickly inspect and reason about what was relevant for API connectivity, instead of manually trying to guess the correct flow.

## Discovery Phase: Finding What Was Needed for Authentication

At this stage, I knew almost nothing about the exact REST API authentication requirements for my use case.

Using Copilot, I worked through the pieces one by one:

1. Identified the REST API client details needed to authenticate.
2. Retrieved the REST API `client_id` used for OAuth2.
3. Confirmed the token endpoint and expected request format.

This was the first major confidence boost. Instead of reading fragmented docs and trial-and-error for hours, Copilot helped me convert clues into a concrete authentication plan.

## Building the Connection: OAuth2 Password Grant

With the `client_id` and credentials ready, the script could request a token from:

- `POST /api/oauth2/token`

The request body used:

- `grant_type=password`
- `client_id=<retrieved client id>`
- `scope=idtoken content`
- username and password

Copilot helped shape this PowerShell call correctly with `Invoke-RestMethod`, including headers and content type.

When the token response came back successfully, that was the turning point: I had a working authenticated connection.

## First Useful API Call: Listing Zones

After authentication, the script used the bearer token to call:

- `GET /api/v3/system/zones`

The query included practical OData parameters (`$count`, `$skip`, `$top`, `$orderby`, `$select`) to keep the response focused and predictable.

Copilot helped draft this URI cleanly, including escaping `$` in PowerShell and adding a cache-buster timestamp to avoid stale responses.

## From Basic Data to Real Output: License Details per Zone

The final improvement was moving from a simple zone list to actionable results.

For each zone, the script performs a follow-up request:

- `GET /api/v3/system/zones/{zoneId}/?$select=id,license`

Then it builds a normalized object with:

- Zone ID
- Zone Name
- License data

This made the output immediately usable for reporting and follow-up automation.

## What Copilot Changed for Me

As a user with limited REST API and OAuth2 experience, Copilot made progress possible in small, understandable steps:

1. It reduced the blank-page problem by suggesting practical next actions.
2. It helped translate technical requirements into working PowerShell code.
3. It accelerated debugging by making endpoint and payload checks easier.
4. It helped me move from "I do not know where to start" to a functioning script that returns real data.

## Practical Outcome

The final script now:

1. Authenticates against Liquit using OAuth2.
2. Retrieves zones from the REST API.
3. Looks up license details per zone.
4. Returns a clean PowerShell object list (`$allLicenses`) that can be exported or filtered.

## Closing Thoughts

This project was less about writing perfect code on day one and more about learning while shipping.

Using GitHub Copilot, I could bridge the gap between limited API knowledge and a real working automation script. If you are in the same position (new to REST and OAuth), start small, validate each step, and let Copilot guide the iteration.

You do not need to be an expert before you begin.
