# League of Legends match duration forecasting

This is a simple project to demonstrate how `creme` may be used to build a "real-time" machine learning app. The idea is to predict the duration of LoL matches using information that is available at the start of the match. Once the match ends, the true duration is used to update the model.

## Screenshots

![home](screenshots/home.png)

![matches](screenshots/matches.png)

![matches](screenshots/match.png)

## Architecture

![architecture](screenshots/architecture.svg)

The goal of this project is to demonstrate that online learning is easy to put in place. Indeed predicting and training are both done inside web requests.

- The machine learning model is stored in [`core/management/commands/add_models.py`](core/management/commands/add_models.py)
- Predictions happen in the `queue_match` function of [`core/services.py`](core/services.py)
- Training happens in the `try_to_end_match` function of [`core/services.py`](core/services.py)
- The average error is computed in the `index` function of [`core/views.py`](core/views.py)

## Usage

### Development

Create an `.env` file with the following structure:

```sh
RIOT_API_KEY=https://developer.riotgames.com/
```

Next, create a local machine named `dev` and connect to it.

```sh
>>> docker-machine create dev
>>> eval "$(docker-machine env dev)"
```

Now you can build the stack.

```sh
docker-compose build
```

You can then start the stack.

```sh
docker-compose docker-compose.dev.yml up -d
```

You only have to build the stack once. However you have to rebuild it if you add or modify a service. You can now navigate to the following pages:

- `localhost:8000` for the app
- `localhost:8082` for [Redis Commander](http://joeferner.github.io/redis-commander/)

Run `docker-compose down` to spin the stack down.

:warning: If you want to delete absolutely everything then run the following command.

```sh
docker container stop $(docker container ls -a -q) && docker system prune -a -f --volumes
```

### Production

Create an `.env` file with the following structure:

```sh
SECRET_KEY=Keep_it_secret,_keep_it_safe
RIOT_API_KEY=https://developer.riotgames.com
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
REDIS_PASSWORD=redis
ADMIN_PASSWORD=creme
```

Run the following command to create a DigitalOcean droplet named `prod`. Replace the variables as you wish (for example `$DIGITALOCEAN_SIZE` could be `s-1vcpu-1gb` and region could be `nyc3`). Run `docker-machine -h` for more details.

```sh
>>> docker-machine create --driver digitalocean
                          --digitalocean-access-token $DIGITALOCEAN_ACCESS_TOKEN
                          --digitalocean-size $DIGITALOCEAN_SIZE
                          --digitalocean-region $DIGITALOCEAN_REGION
                          prod
```

You can now run `docker-machine ls` to see the instance you just created. Next run the following commands to deploy the app.

```sh
>>> eval "$(docker-machine env prod)"
>>> docker-compose build
>>> docker-compose up -d
```

Finally run `docker-machine ip prod` to get the IP address of the production instance. If you want to check out the logs run `docker-compose logs --tail=1000`.

For more information about deploying a Django app with Docker check out [this](https://realpython.com/django-development-with-docker-compose-and-machine/) down to earth post.
