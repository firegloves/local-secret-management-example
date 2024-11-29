# Local Secret Manager example

## Overview

This example project shows how to set up and use a Docker instance of Infisical to centralize secrets and remove them from the .env files.
This solution tries to solve 2 problems:
- having confidential data stored within projects, with the risk of sharing it
- having configuration files scattered throughout the operating system

## Prerequisites

- Docker installed
- NodeJS installed

### Most relevant files

`.env`: contains local environment variables
`.infisical.json`: (to be created by the Infisical CLI) contains references to the Infisical instance to be used. This file should not be shared (or pushed), but since it points to a local Infisical instance, its leak can't affect anything unless the attacker has direct access to the user's machine
`package.json`: contains 2 scripts where
    - Infisical retrieves all the secrets in its target instance (depending on the required dev or prod environment)
    - and injects them as environment variables
    - starts the application
`src/app/page.tsx`: contains a chunk that prints the value of the `MY_SECRET` environment

Secrets retrieved by Infisical overwrite the variables of the .env values. For secrets non present in Infisical, a fallback will look in the .env file

## Environment configuration

### Infisical project configuration

Since we want to rely on a local instance of Infisical, we must configure it:

1. Create a new directory (e.g. `local-infisical`) and enter it
2. Install the Infisical CLI by running `brew install infisical/get-cli/infisical` (don't use OS X? [Here](https://infisical.com/docs/cli/overview) you can find specific instructions)
3. Download the Infisical docker compose file by running `curl -o docker-compose.prod.yml https://raw.githubusercontent.com/Infisical/infisical/main/docker-compose.prod.yml` (or download the file in any other way)
4. Download the `.env.example` file containing a set of defaults to run Infisical `curl -o .env https://raw.githubusercontent.com/Infisical/infisical/main/.env.example`
5. Start the Docker daemon
6. Start Infisical by running `docker-compose -f docker-compose.prod.yml up`

Steps 1 to 4 only have to be performed once during the Infisical configuration phase and can be skipped at subsequent times.

References:
- https://infisical.com/docs/self-hosting/deployment-options/docker-compose

### Secrets configuration

1. Access your Infisical instance by opening your browser at http://localhost and signup
2. An `Org Admin` should now have been automatically created for you.
3. Select `Overview` from the menu on the left
4. Add a new project
5. Create a new secret named `MY_SECRET`. You can choose any value you like. Check the check box of the `Development`

### Application configuration

It is now necessary to configure the Infisical CLI so that it can retrieve secrets from the Infisical instance.

1. Log in to the Infisical instance by running `infisical login`.
2. Choose `hosted or dedicated instance`.
3. Set the domain address to `http://localhost`.
4. Follow the instructions printed from the shell to log into the browser and copy the generated token.
5. Now open a shell in the root folder of this project
6. Initialise Infisical by running `infisical init`.
7. Choose the desired organisation from the list
7. Choose the desired project from the list

At this point the `.infisical.json` file have been created.

References:
- https://infisical.com/docs/cli/usage

## Starting the application

To start the application, we can take advantage of the `npm` and `package.json` scripts.
Focus on these two in particular:

```json
"inf-dev": "infisical run --env=dev/ -- next dev",
"inf-prod": "infisical run --env=prod -- next dev",
```

Both run the Infisical CLI to retrieve secrets from the Infisical instance. Then these secrets will be injected into the application (started by the last part of the script).
Focus on `--env`. This tells the Infisical CLI the environment from which to retrieve the secrets. If you remember, in step 5 of the secrets configuration steps, we only created one secret for the development environment.
So if you start the application with `npm run inf-dev` and open the resulting URL, you will see the value retrieved from Infisical.
Otherwise, if you start the application using `npm run inf-prod` and open the resulting URL, you will see the value in the `.env` file.

## Final considerations

Using this approach, we can concentrate on simplifying the development process, without affecting the deployment flows already in place for production.
The main disadvantage is that we must have docker up and running, unless we decide to have a centralised instance of Infisical, but this would render the solution useless, because we would be forced to hide the `infisical.json`

### Next steps

If we decide to adopt this solution, we could write a script to maximise the automation of the configuration of user systems and Infisical instances.
Moreover, a good idea would probably be to use a port other than 80