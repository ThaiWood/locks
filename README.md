# Locks

Locks is a virtual machine traffic control service for SauceLabs users. 

It ensures a test is never waiting to acquire a virtual machine. This allows orchestration software to avoid killing tests early for reasons unrelated to failure (i.e. a test should never be punished for running too long if it was merely waiting for a VM to become available).

## Installing

The preferred method for installing `locks` is as a global module:

```
npm install -g testarmada-locks
```

## Running

```
export SAUCE_ACCESS_KEY='xxxxxxxxxxxxxxxxxxxxxxxxxxxxxx'
export SAUCE_USERNAME='xxxxxxxxxxxx'

# optionally configure statsd
export LOCKS_STATSD_URL=hostname.of.statsd.server.com
export LOCKS_STATSD_PREFIX=locks

# optionally configure locks' listener port (default 4765)
export LOCKS_PORT=4567

locks
```

## Running with Docker

Check out our Dockerfile [here](./Dockerfile)

```
# docker build . -t testarmada/locks

# docker run -e SAUCE_ACCESS_KEY='YOUR_KEY' \
-e SAUCE_USERNAME='YOUR_USER' \
--restart=always \
--publish=4765:4765 \
--detach=true \
--name=locks \
-d testarmada/locks
```

## Websocket API

To communicate with `locks`, open a websocket to your locks URL and use the claim and release API, outlined below:

### Claiming VMs

Send the following to `locks` to attempt to claim a VM:

```json
{
  "type": "claim"
}
```

If `locks` accepts your claim, you will eventually receive the following message:

```json
{
  "accepted": true,
  "token": <string>
}
```

The `locks` websocket API will always try to satisfy as many claims as you've recently made until it runs out of capacity. If this happens, your client will receive a rejection. If a claim is rejected, the return message will look like this:

```json
{
  "accepted": false
}
```

If your claim is rejected, `locks` is no longer attempting to claim a VM for you, and your client must poll again.

### Releasing VMs (OPTIONAL)

To more accurately track VM supply, your client can send a courtesy message informing `locks` that you are no longer using a recently-claimed VM. Simply send a release message along with the token of the claim. Note that this is a safe operation even if `locks` has already expired the corresponding claim token.

```json
{
  "type": "release",
  "token": <string>
}
```

### Proxy Configuration

To use a proxy to reach Saucelabs when querying the Saucelabs API, `locks` checks the presence of an environment variable called `SAUCE_OUTBOUND_PROXY` in the sauce provider.

```
$ export SAUCE_OUTBOUND_PROXY=http://your-internal-proxy-host:8080
$ locks
```

