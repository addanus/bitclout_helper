# in case need to build from a local backend
# this file would be located here: bitclout/run/backend/Dockerfile
# and would be referenced when running bitclout/run/run.sh - or more precisely:
# $ cd bitclout/run; docker-compose build; docker-compose down --remove-orphans; docker-compose up -d

FROM docker.io/bitclout/backend:latest
WORKDIR /bitclout/bin
CMD ./backend run
