# choose the base, in our case debian or alpine, in penultimate version
FROM		alpine:latest

# set the working dir, ensuring that the folder is created
WORKDIR		/run

# add entrypoint
COPY		--chown=root:root --chmod=744 entrypoint /run

# set the program launched when `docker run`
ENTRYPOINT	[ "/run/entrypoint" ]
