
## Backup & Restore Server

- Masuk di bash docker container

  ```sh
  docker exec -it [container_id] bash
  ```

- Backup database postgres

  ```sh
  pg_dump -U [user_database] -d [database_name] -f [file_name.sql]
  ```

- Copy file dari container ke local

  ```sh
  docker cp [container_name]:[dir/file_name] [file_name]
  ```

- Copy file yang sudah di backup ke container target

  ```sh
  docker cp [file_name] [container_name]:[dir/file_name]
  ```

- Masuk ke bash docker container id

  ```sh
  docker exec -it [container_id] bash
  ```

- Restore file ke database

  ```sh
  psql -U [user_database] -d [database_name] < [file_name]
  ```
