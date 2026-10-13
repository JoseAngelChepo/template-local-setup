# MongoDB Atlas — template setup

Choose Atlas when the user asks for it or provides an Atlas connection. Do not
install MongoDB, start a local daemon, or require port 27017 for this route.

## Configure

1. Prepare `template-services/.env` with `npm run setup-env`. Preserve an existing
   file; if the user will enter the URL, leave `MONGODB_URI=` and tell them the
   exact file path. Never echo the connection value or request passwords in chat.
2. The user provides a database user and a connection string containing a
   dedicated application database. Atlas console users and database users are
   different identities. Reserved characters in credentials must be URL-encoded.
3. Atlas Network Access must allow the IP of the machine that runs the backend.
   Use the relevant IP, not a blanket `0.0.0.0/0` rule. If the API runs elsewhere,
   authorize that host's outbound IP.
4. Install app dependencies with `npm ci`, start the backend, and check
   `GET http://localhost:3001/api/v1/health`. Continue runtime smoke tests only
   after it returns `status: ok` and `database.status: up`.

[Atlas connection guide](https://www.mongodb.com/docs/atlas/connect-to-database-deployment/)

## Troubleshooting

- **Server selection timeout / TLS alert:** check that the cluster is running,
  the backend machine's IP is permitted, and outbound networking is available.
  A TLS alert alone does not prove which of those conditions is responsible.
- **Authentication failed:** verify the database user, password encoding, and
  permissions for the selected application database.
- **SRV/DNS failure:** verify the cluster hostname and DNS resolution on the
  backend host.
- **Certificate error:** inspect the trust store/system time; do not disable TLS
  verification to make the connection succeed.

Report only necessary error types/codes and redact connection strings. Builds
and typechecks can proceed while the user finishes connection setup; label
runtime verification as pending until the health check succeeds.

[Atlas connection troubleshooting](https://www.mongodb.com/docs/atlas/troubleshoot-connection/)
