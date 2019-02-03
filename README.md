<img src="https://s3.amazonaws.com/devmountain/readme-logo.png" width="250" align="right">

# Sentry Guide

A guide on how to integrate [Sentry](https://sentry.io/welcome/) into new projects.

## Setting up the Digital Ocean Droplet

- Install `sentry-cli` globally:
  - `curl -sL https://sentry.io/get-cli/ | bash`
- Verify installation with `sentry-cli --version`.
- Create a `.sentryclirc` in the home (`~`) directory:
  - <details>
    <summary> <code> .sentryclirc </code> </summary>
    <br />

    ```yaml
    [auth]
    token='slack dev member for this token'
    ```

    </details>

## Adding configuration variables to your project

- Create a `.env` file or add to an existing `.env` file:
  - <details>
    <summary> <code> .env </code> </summary>
    <br />
    
    ```env
    SENTRY_AUTH_TOKEN='slack dev team member for this token'
    SENTRY_URL='https://sentry.devmountain.com/'
    SENTRY_ORG='devmountain'
    SENTRY_PROJECT='must match name on project in Sentry dashboard'
    SENTRY_ENVIRONMENT='development || staging || production'
    ```
    
    </details>
- Create a `deployWithSentry.sh` script:
  - <details>
    <summary> <code> deployWithSentry.sh (for basecamphub) </code> </summary>
    <br />
    
    ```sh
    # import environment variables
    export $(xargs < /home/devmtn/basecamp-hub/.env)

    # Creates a new release in sentry based off the most recent git commit sha
    sentry-cli releases new $(sentry-cli releases propose-version)

    # Deploy
    /home/devmtn/basecamp-hub/deploy.sh

    # Install Dependencies & Rebuild
    cd /home/devmtn/basecamp-hub/public && yarn install --production=false && yarn build

    # Define where the maps are
    cd /home/devmtn/basecamp-hub/public/dist/static/js

    printf "\n//# sourceMappingURL=app.js.map" >> app.js
    printf "\n//# sourceMappingURL=manifest.js.map" >> manifest.js
    printf "\n//# sourceMappingURL=vendor.js.map" >> vendor.js

    # Finalize the release and upload the sourcemaps
    sentry-cli releases finalize $(sentry-cli releases propose-version)
    sentry-cli releases files $(sentry-cli releases propose-version) upload-sourcemaps /home/devmtn/basecamp-hub/public/dist/static/js --ext map

    # Deploy the release
    sentry-cli releases deploys $(sentry-cli releases propose-version) new -e $SENTRY_ENV
    ```
    
    </details>

## Configuring your project - Front End

- Bundle names must not have hashes in them so Sentry can successfully create source maps.
  - In Vue, this is done by going into `webpack.prod.conf.js` and changing the `output` to not use `[hash]` in the filename.
- Raven must be configured before the application is initialized.
  - In Vue, this is done by going into `main.js`:
    - <details>
      <summary> <code> public/src/main.js </code> </summary>
      <br />
      
      ```js
      import Raven from 'raven-js'
      import RavenVue from 'raven-js/plugins/vue'

      // Initialize Raven Config before Vue App
      Raven.config(process.env.RAVEN_CONFIG_URL, { environment: process.env.NODE_ENV })
        .addPlugin(RavenVue, Vue)
        .install()
      ```
      
      </details>
