# operation

Contains the files to orchestrate the entire restaurant review app--including the model service and frontend application. Currently this is just a `docker compose`-based setup. Later on `kubernetes` will be used as well.

## Running

In order to run, use the following command. Omitting the the `-d` makes `docker compose` take over your current shell instead of running as a daemon in the background.

```bash
docker compose up -d
```

To view the logs of a currently-running application use the following command. Omitting `-f` means that `docker` will only print out the current logs and exit, rather than showing logs as they appear.

```bash
docker compose logs
```

To stop everything run 

```bash
docker compose down
```

In case you would like to update the versions of images that the `compose` setup uses, run the following command:

```bash
docker compose pull
```

## Code structure

    .
    ├── model-training                    # Code to pre-process and train the review classifier model
    ├── model-service                     # Flask service to serve the model
    ├── lib                               # Library that the frontend app depends on. Currently no functionality.
    ├── app                               # Frontend app that displays a UI and allows for a user to query the model-service
    └── operation                         # Orchestration code to run the app and service, currently just a docker-compose setup.
