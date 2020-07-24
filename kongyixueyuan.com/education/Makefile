.PHONY: all dev clean build env-up env-down run

all: clean build env-up run

dev: build run

##### ENV
env-up:
	@echo "Start environment ..."
	@cd fixtures && docker-compose up --force-recreate -d
	@echo "Environment up"

##### BUILD
build:
	@echo "Build ..."
	@go build
	@echo "Build done"

##### RUN
run:
	@echo "Start app ..."
	@./education

env-down:
	@echo "Stop environment ..."
	@cd fixtures && docker-compose down
	@echo "Environment down"

##### CLEAN
clean: 
	@echo "Clean up ..."
	@rm -rf /tmp/kongyixueyuan-* kongyixueyuan
	@docker rm -f -v `docker ps -a --no-trunc | grep "kongyixueyuan" | cut -d ' ' -f 1` 2>/dev/null || true
	@docker rmi `docker images --no-trunc | grep "kongyixueyuan" | cut -d ' ' -f 1` 2>/dev/null || true
	@echo "Clean up done"
