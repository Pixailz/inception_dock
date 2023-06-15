SHARED_DIR		:= Shared

COMPOSE			:= docker compose

# if TARGET is not specified, can be blank or a service name
TARGET			?=

# macro to make dir, '=' not ':=' as '=' will evaluate '$(1)' when called, ':='
# will do it only once
MKDIR			= [ ! -d $(1) ] && mkdir -p $(1)

.PHONY: all up run build re clean

all:			up

up:				build
	$(COMPOSE) up $(TARGET)

run:			build
	$(COMPOSE) run -it $(TARGET)

build:			$(SHARED_DIR)
	$(COMPOSE) build $(TARGET)

kill:
	$(COMPOSE) kill $(TARGET)

down:			kill
	$(COMPOSE) down

fclean:			down
	rm -rf $(SHARED_DIR)

re:				kill all

$(SHARED_DIR):
	$(call MKDIR,$(@))
