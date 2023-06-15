# choose the base, in our case debian or alpine, in penultimate version
FROM		alpine:latest

# install package needed
RUN			apk add --update \
				python3 \
			&& rm -rf /var/apk/cache/*

# set the working dir, ensuring that the folder is created
WORKDIR		/run

# copy from host to container
## --chown to own with a specific user
## --chmod to change the permission on the file
COPY		--chown=root:root --chmod=644 index.html /run

# set the program launched when `docker run`
ENTRYPOINT	[ "python3" ]

# set the argument passed to that program
CMD [ "-m", "http.server", "-d", "/run" ]
