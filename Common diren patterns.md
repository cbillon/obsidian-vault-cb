---
tags:
  - direnv
link: https://oneuptime.com/blog/post/2026-03-02-how-to-install-and-configure-direnv-on-ubuntu/view
---
# Common .envrc Patterns
## Load environment-specific config
```
cat > .envrc <<'EOF'

ENV=${ENV:-development}

case "$ENV" in
  development)
    export DATABASE_URL="postgresql://localhost/myapp_dev"
    export LOG_LEVEL="debug"
    ;;
  staging)
    export DATABASE_URL="postgresql://staging-db/myapp_staging"
    export LOG_LEVEL="info"
    ;;
  *)
    echo "Unknown ENV: $ENV"
    exit 1
    ;;
esac

export APP_ENV="$ENV"
EOF

direnv allow
```
```

```

Load staging config
```
ENV=staging direnv reload
```

Or set ENV before entering the directory:
```
export ENV=staging
cd ~/test-project
```
### Using a .env File with direnv[](https://oneuptime.com/blog/post/2026-03-02-how-to-install-and-configure-direnv-on-ubuntu/view#using-a-env-file-with-direnv)

Many projects already have a `.env` file (used by Docker Compose, Python dotenv, etc.). Load it with direnv:
```
cat > .envrc <<'EOF'
# Load variables from .env file if it exists
if [ -f .env ]; then
  dotenv
fi
EOF

# Create a .env file
cat > .env <<'EOF'
DATABASE_URL=postgresql://localhost/myapp
SECRET_KEY=my-secret-key
EOF

direnv allow
# direnv will now load .env automatically
```
