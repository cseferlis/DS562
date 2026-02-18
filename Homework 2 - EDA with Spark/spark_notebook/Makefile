up:
	docker compose up -d

down:
	docker compose down

restart:
	docker compose down -v
	docker compose build --no-cache
	docker compose up -d

.PHONY: up down restart
