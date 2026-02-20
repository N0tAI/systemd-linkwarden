# Linkwarden Deployment

My personal containerized setup for [Linkwarden](https://github.com/linkwarden/linkwarden) using Quadlet units managed directly via systemd.

## Setup

This setup will assume that you want to run this as your own user, if you want to run it as root (otherwise unmodified) you should be good to just change `${XDG_CONFIG_HOME}` to `/etc` and omitting the user flag of systemctl (and either enter your root password when prompted or prefix with sudo/doas).

1. Copy the repository files over to `${XDG_CONFIG_HOME}/containers/systemd/`
2. Create your secrets, the containers require a fair number
    - `linkwarden_nextauth`: Your [`NEXTAUTH_SECRET`](https://docs.linkwarden.app/self-hosting/installation#2-configure-the-environment-variables)
    - `linkwarden_meili_master_key`: Your [`MEILI_MASTER_KEY`](https://docs.linkwarden.app/self-hosting/installation#2-configure-the-environment-variables)
    - `linkwarden_postgres_password`: Your [`POSTGRES_PASSWORD`](https://docs.linkwarden.app/self-hosting/installation#2-configure-the-environment-variables)
    - `linkwarden_llm_api_key`: Your [OpenRouter API](https://openrouter.ai/settings/keys) Key
    - `linkwarden_db_url`: The full url (relative to the pod) to access PostgreSQL in the form of `postgresql://<user>:${linkwarden_postgres_password}@localhost:<postgres port>/<db>`
3. Reload the service manager: `systemctl --user daemon-reload`
4. Start linkwarden: `systemctl --user start linkwarden`

## Architecture

I chose to deploy this with systemd-native Podman Quadlet units rather than Docker Compose for the following reasons:

- **Less dependencies:** by managing the units directly with systemd I skip the need for an intermediary monitoring daemon/script leaving less room for failure to launch/work.
- **Service-like Behaviour:** You can treat these containers like any other systemd service, the write logs via `journalctl`, are controlled by `systemdctl`, and can properly integrate with targets and other services outside of the stack.
- **Rootless:** I can run this entire stack without requiring root, allowing me to deploy it where I want without risking an security flaws that may result in privilege escalations.
- **Network Isolation:** by default services behave as if they're on the same machine but are still able to only have the necessary ports exposed outside of their private network.
- **Easier to pass secrets:** Just that, it's a one-liner to set specific environment variables with quadlet units vs the roundabout way of docker-compose (secret -> file -> read to env var).

## Services

1. **Linkwarden:** The main application exposed over localhost:3000
2. **Meilisearch:** Indexing/Search capabilities. (Running dev mode for ease of integration, restricted solely to the pod's internal network)
3. **PostgreSQL:** The Database, restricted to the pod's internal network
