[![Gitter Chat](https://img.shields.io/badge/gitter-join%20chat-brightgreen.svg)](https://gitter.im/CiscoSecurity/Threat-Response "Gitter Chat")

# Recorded Future Relay (Cisco Hosted)

A Cisco SecureX Relay implementation using [Recorded Future](https://www.recordedfuture.com/) as a third-party Cyber Threat
Intelligence service provider.

The Relay itself is just a simple application written in Python that can be
easily packaged and deployed in docker container.

The code is provided here purely for educational purposes.

## Rationale

- We need an application that will translate API requests from SecureX Threat Response to the third-party integration, and vice versa.
- We need an application that can be completely self contained within a virtualized container using Docker.

## Testing (Optional)

Open the code folder in your terminal.
```
cd code
```

If you want to test the application you have to install dependencies from the [Pipfile](code/Pipfile) file:
```
pip install --no-cache-dir --upgrade pipenv && pipenv install --dev
```

You can perform two kinds of testing:

- Run static code analysis checking for any semantic discrepancies and
[PEP 8](https://www.python.org/dev/peps/pep-0008/) compliance:

  `flake8 .`

- Run the suite of unit tests and measure the code coverage:

  `coverage run --source api/ -m pytest --verbose tests/unit/ && coverage report`

**NOTE**. If you need input data for testing purposes you can use data from the [observables.json](code/observables.json)
### Building the Docker Container
In order to build the application, we need to use a `Dockerfile`.  

 1. Open a terminal.  Build the container image using the `docker build` command.

```
docker build -t tr-05-recorded-future .
```

 2. Once the container is built, and an image is successfully created, start your container using the `docker run` command and specify the name of the image we have just created.  By default, the container will listen for HTTP requests using port 9090.

```
docker run -dp 9090:9090 --name tr-05-recorded-future tr-05-recorded-future
```

 3. Watch the container logs to ensure it starts correctly.

```
docker logs tr-05-recorded-future
```

 4. Once the container has started correctly, open your web browser to http://localhost:9090.  You should see a response from the container.

```
curl http://localhost:9090
```

## Implementation Details

This application was developed and tested under Python version 3.9.

### Implemented Relay Endpoints

- `POST /health`
  - Verifies the Authorization Bearer JWT and decodes it to restore the original credentials.
  - Authenticates to the underlying external service to check that provided credentials are valid and the service is available at the moment. 

- `Post /observe/observables`
  - Accepts a list of observables.
  - Verifies the Authorization Bearer JWT and decodes it to restore the original credentials.
  - Makes a series of requests to the underlying external service to query for some cyber threat intelligence data on each supported observable.
  - Maps the fetched data into appropriate CTIM entities.
  - Returns a list per each of the following CTIM entities (if any extracted):
    - Verdict,
    - Judgement,
    - Indicator,
    - Sighting,
    - Relationship
   
- `POST /refer/observables`
  - Accepts a list of observables and filters out unsupported ones.
  - Builds a search link per each supported observable to pivot back to the
  underlying external service and look up events with the observable there.
  - Returns a list of those links.
  
- `POST /deliberate/observables`
  - Accepts a list of observables and filters out unsupported ones.
  - Verifies the Authorization Bearer JWT and decodes it to restore the
  original credentials.
  - Makes a series of requests to the underlying external service to query for
  some cyber threat intelligence data on each supported observable.
  - Maps the fetched data into appropriate CTIM entities.
  - Returns a list per each of the following CTIM entities (if any extracted):
    - `Verdict`.
  
- `POST /version`
  - Returns the current version of the application.
  
### Supported Types of Observables

- `ip`
- `ipv4`
- `ipv6`
- `sha1`
- `sha256`
- `md5`
- `domain`
- `url`

### CTIM Mapping Specifics

Each response from the Recorded Future API for the supported observables generates the following CTIM entities:
- `Judgement`
- `Indicator`
- `Sighting` of `Indicator`
- `Sighting` of `Observable`
- `Relationship`
- `Verdict` 
  
- `Judgement`, `Indicator`, `Verdict`, `Sighting` of `Indicator` are based on `.rules[]`.
- `Sighting` of `Observable` is taken from `.sightings[]`.
 
Relationships are the following: 
- `Judgement` -> element-of `Indicator`
- `Sighting` -> member-of `Indicator` (for Sighting of an Indicator)

Test PR created by SRE bot
