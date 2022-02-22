# mr_redis

MR_Redis is an MQTT5 Redis-based server.

It provides a Redis module (mr_redis.so) that implements MQTT5 over RESP (REdis Serialization Protocol).

The module should be loaded into Redis, typically via the Redis config file or optionally via a Redis CLI command.

The server uses the MisteR MQTT5 library, incorporated as a git submodule, to validate, pack & unpack MQTT5 packets.

The project is in an initial non-working state.
