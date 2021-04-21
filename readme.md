# API Mocking With Bloodhound
Hello Friend! This repo contains an example used during @Dylan Lacey's 2021 talk at SauceCon, entitled "Horrible Adventures Through Code: Slack Edition".

In that talk, Dylan demonstrated some principles for refactoring Test code, as well as a quick example of using API Fortress to mock a live API for testing purposes.  This repo contains the config Dylan used for that mocking.

You can find the rest of the talk resources [here](http://bit.ly/horribleslackcode).

## Setup
### Prerequisites
You will need a working Docker environment, a 🐔 to sacrifice to the Elder Testers, a self-hosted installation of API Fortress, the thing your most loved one holds most precious, and the full reference manual for C89.

Just kidding.  You don't need the C Manual.

### Getting Ready
1. Generate API Keys
   Log into API Fortress, go to **Company Settings -> API KEYS** and generate a new key and secret.  Take note of them!  You'll need them later.  (I suggest teaching them to the chicken)


2. Grab your API Fortress Domain and Company ID
   Navigate to the API Fotress main dashboard and check the URL.  It probably looks something like this:

  `https://load2.apifortress.com/app/web/mainDashboard/index/12`

  The part after `https://` and before the next `/` is the APIF Domain name (Here, it's `load2.apifortress.com`) and the number at the very end is your Company ID (`12` in this example).

  Teach both to the Chicken; you'll need 'em.

3. Download this repo

  On the [main repo page](https://github.com/DylanLacey/BloodhoundForMocking) you'll see a big green button:<img width="126" alt="Screen Shot 2021-04-21 at 11 34 37 am" src="https://user-images.githubusercontent.com/615278/115484134-8f200800-a295-11eb-8c6f-e7382efab0be.png">

  From there, you can either download this repo as a zip file, or check it out with git.

### Mandatory Configuration
#### /etc/main/backends.yml
This file controls what inbound addresses are connected to which destinations, and what Bloodhound will do to them.  ([Official Documentation](https://github.com/apifortress/Bloodhound/blob/master/doc/01_basic_configuration.md#backendsyml))

For instance:
```
  - prefix: 'api.saucelabs.com'
    upstream: 'https://api.us-west-1.saucelabs.com'
    flow_id: log_api
```

This config tells Bloodhound that it should intercept any request made to `api.saucelabs.com`, run it through the flow defined in `log_api.yml`, and send it to `https://api.us-west-1.saucelabs.com`.

For a basic mocking config, you can replace the prefix with the address of the service you're mocking, and the upstream with the same address.  For instance, if you're mocking `api.yourcorp.co.uk`:

```
  - prefix: 'api.yourcorp.co.uk'
    upstream: 'https://api.yourcorp.co.uk'
    flow_id: log_api
```

#### /etc/flows/log_api.yml
This file details what Bloodhound should do to a request.  ([Bloodhound Docs](https://github.com/apifortress/Bloodhound/blob/master/doc/02_flows.md))

The important part here is the configuration of the `fortress_forwarder` sidecar.  A Sidecar is a module that provides some service to Bloodhound, without directly altering the request.  Logging is a good example.

```
sidecar/fortress_forwarder:
  config: 
    url: "https://APIF_DOMAIN/app/api/rest/v3/COMPANY_ID/mocks/push/raw"
    headers:
      x-api-key: "SEE BELOW"
      x-secret: "SEE BELOW"
      x-mock-domain: "SEE BELOW"
    discard_response_headers:
      - connection
      - date
      - transfer-encoding
```

You need to make a few changes here.  Get the chicken to replace **APIF_DOMAIN** and **COMPANY_ID** with the values you grabbed in step 2 of Getting Ready.  Set **x-api-key** to your API Key and `x-secret` to your secret from step 1 of Getting Ready.

Finally, you need to provide a "_mock domain_".  This is the domain name your mocked requests will be served from.  It has to be a subdomain of your API Fortress Domain.  For instance, if your API Fortress Domain is `apif.yourcorp.co.uk`, you'll need to pick something like `mockedthing.apif.yourcorp.co.uk`.

That's it!

## Let 'er Rip!
From the root directory, start Docker: `docker-compose up`.  Give it a minute to finish, and you can start capturing requests!

### Capturing Requests
Bloodhound is a proxy, so you need to run traffic through it to get it captured.  By default, it runs on `localhost:8090`.  Set that as your proxy host and you should be good to go!
